# Contratos Versionados entre Sistemas: v1 → v2

*Notas de estudio a partir del análisis de `gentle-pi`*

---

## 1. El problema de fondo: dos sistemas que evolucionan por separado

`gentle-pi` (el plugin de Pi) y `gentle-ai` (el runtime nativo que hace la revisión) son **dos códigos distintos, publicados y versionados por separado**, que se comunican a través de una API. Esto es exactamente la misma situación que un frontend y un backend, o un microservicio y otro: cada uno se puede actualizar de forma independiente, en momentos distintos, potencialmente por equipos distintos.

El problema clásico que aparece acá: **¿qué pasa si un lado cambia el formato de los datos que manda o espera, y el otro lado todavía no se actualizó?**

Sin ningún control, esto rompe en producción de la forma más silenciosa y peligrosa posible: no con un error claro, sino con datos mal interpretados. Un campo que antes era un string y ahora es un objeto, un campo que se eliminó y el consumidor sigue esperando, un campo nuevo obligatorio que el emisor viejo nunca manda.

---

## 2. Qué es un "contrato" en este contexto

Un **contrato** es la especificación formal de la forma exacta que deben tener los mensajes que dos sistemas intercambian — típicamente un **JSON Schema**, un archivo `.proto` (Protocol Buffers), o un esquema OpenAPI. No es documentación informal ("mandale un JSON con estos campos más o menos") — es una definición **verificable mecánicamente**.

```json
// Ejemplo simplificado de lo que sería un fragmento de contrato v1
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["candidate_diff", "lineage"],
  "properties": {
    "candidate_diff": { "type": "string" },   // v1: diff en base64 inline
    "lineage": { "type": "string" }
  }
}
```

`gentle-pi` llama a esto `review-integration/v1` y `review-integration/v2` — cada uno es un contrato negociado, con schemas "byte-identical" (idénticos byte a byte) guardados en ambos lados del sistema y **hash-checked antes de empaquetar** (`contracts/review-integration/v1/`, `contracts/review-integration/v2/`).

Ese detalle — hash-checked antes de packaging — es clave: no alcanza con que el contrato "esté documentado", tiene que haber una verificación automática de que el código que consume el contrato coincide exactamente con la versión del contrato que dice implementar. Es lo mismo que hacíamos con CAS: la integridad se garantiza matemáticamente, no por convención.

---

## 3. Qué cambió entre v1 y v2 (y por qué es un buen ejemplo de evolución de schema)

El repo lo describe con precisión:

> "Gentle AI también publica un segundo contrato negociado, `gentle-ai.review-integration/v2`, que reemplaza el transporte Base64 `candidate_diff` por `base_tree`/`candidate_tree` inmutables más un `changed_path_manifest` ordenado, y nunca un patch inline."

Traducido a un cambio concreto:

| | v1 | v2 |
|---|---|---|
| **Cómo se transmite el cambio** | Un diff completo codificado en Base64, embebido inline en el mensaje | Referencias inmutables a los árboles (`base_tree`, `candidate_tree`) + una lista ordenada de rutas cambiadas |
| **Tamaño del payload** | Crece con el tamaño del diff — puede ser enorme | Crece con la cantidad de archivos, no con el contenido — mucho más liviano |
| **Verificabilidad** | El consumidor tiene que parsear y confiar en el diff tal cual llega | El consumidor puede resolver cada árbol de forma independiente contra Git y verificar que corresponde a lo que dice el manifiesto |

Esto es un ejemplo real de una motivación típica para versionar un contrato: **el formato viejo tenía una limitación de diseño** (payloads gigantes, menor verificabilidad) que el nuevo formato corrige, pero corregirla implica un cambio **incompatible hacia atrás** (breaking change) — un consumidor v1 no puede simplemente ignorar el campo nuevo y seguir funcionando, porque el campo que necesita (`candidate_diff`) directamente no está más.

---

## 4. Por qué no basta con "actualizar los dos lados al mismo tiempo"

En un monolito, cambiar un formato de datos es fácil: cambiás el código, corre el build, listo, todo el sistema usa la versión nueva de forma atómica. En un sistema de **múltiples binarios independientes que se instalan y actualizan por separado**, esa atomicidad no existe. `gentle-pi` (versión X) puede estar corriendo contra un `gentle-ai` (versión Y) que se instaló en un momento distinto, con su propio ciclo de release.

Esto es el problema clásico de **compatibilidad de versiones en sistemas distribuidos**, y hay tres estrategias típicas para resolverlo:

1. **Big bang**: todos los consumidores se actualizan exactamente al mismo tiempo que el productor. Frágil, casi nunca es realista fuera de sistemas muy chicos controlados por un solo equipo.
2. **Compatibilidad hacia atrás perpetua**: el productor nunca rompe el contrato viejo, solo agrega campos opcionales. Simple pero acumula deuda técnica indefinidamente — nunca podés sacarte de encima un formato viejo.
3. **Negociación de versión explícita + ventana de coexistencia**: ambos contratos (v1 y v2) existen simultáneamente durante un período, cada mensaje se identifica con qué versión de contrato usa, y hay una migración planeada con fecha de corte. Es la estrategia más trabajosa pero la más robusta.

`gentle-pi` usa la tercera, con una decisión interesante: **no mantienen un "dual-lane" permanente**. El README dice explícitamente que `gentle-pi` está migrando *solo* a `/v2`, sin fallback dual, y que el corte (*cutover*) está condicionado (*gated*) a que el runtime fijado confirme que sirve el contrato v2. Es decir, usan la estrategia 3 pero con un plan explícito de terminar en la estrategia 1 (todo en v2) una vez que la migración se completa — no la mantienen como coexistencia eterna.

---

## 5. Negociación de capacidades: cómo se decide qué versión usar en runtime

Un detalle importante del diseño es que la versión de contrato a usar **no está hardcodeada** — se negocia dinámicamente:

> "Capabilities are cached by that executable digest."

Esto quiere decir: cuando `gentle-pi` arranca, resuelve el binario exacto de `gentle-ai` instalado, calcula su hash (de nuevo, CAS — el mismo patrón que ya vimos), y a partir de ese hash específico sabe/cachea qué capacidades (qué versiones de contrato) ese binario exacto soporta. No asume una versión fija — la deriva del binario real presente en el sistema.

Esto es un patrón conocido como **capability negotiation** (negociación de capacidades), muy usado en protocolos de red reales:

- **TLS handshake**: cliente y servidor negocian qué versión de TLS y qué cifrados van a usar, basado en lo que cada uno soporta.
- **HTTP `Accept` / `Content-Type` headers**: el cliente le dice al servidor qué formatos puede entender, y viceversa.
- **gRPC / Protocol Buffers**: usan reflection para que un cliente pueda preguntarle a un servidor qué métodos y versiones soporta antes de invocar nada.

La idea común: en vez de **asumir** una versión compartida, el sistema la **verifica explícitamente** antes de comunicarse — coherente con el patrón fail-closed que vimos en los otros temas.

---

## 6. Contract Testing: verificar la promesa, no solo documentarla

**Contract testing** es una práctica de testing donde, en vez de (o además de) tests de integración end-to-end costosos, cada lado del contrato tiene tests que verifican **exclusivamente que cumple su parte del contrato**, independientemente del otro sistema.

- El *productor* (`gentle-ai`) tiene tests que verifican: "cualquier mensaje que yo mande cumple el schema v2 publicado".
- El *consumidor* (`gentle-pi`) tiene tests que verifican: "yo puedo procesar correctamente cualquier mensaje válido según el schema v2, y rechazo correctamente cualquier mensaje que no lo cumpla".

`gentle-pi` menciona **"conformance fixtures"** (`contracts/review-integration/v1/`, `.../v2/`) — casos de prueba concretos y fijos que sirven como *ground truth* de "esto es un mensaje válido según este contrato exacto". Son ejemplos reales de payloads, no solo la definición abstracta del schema, contra los cuales se puede correr un test automatizado tanto del lado productor como del lado consumidor.

Este enfoque evita el problema clásico de los tests de integración tradicionales: no necesitás tener *ambos* sistemas corriendo al mismo tiempo para verificar que son compatibles — cada uno se testea contra el contrato compartido de forma independiente, y **si ambos pasan sus tests contra el mismo contrato, están garantizados a ser compatibles entre sí sin necesidad de correrlos juntos**.

---

## 7. Un detalle sutil: nombres que coinciden mal

El repo hace una aclaración explícita que vale la pena notar como lección de comunicación técnica:

> "Este contrato del proveedor `v2` no está relacionado con el naming interno de autoridad de revisión 'compact-v2' de Pi — el dígito compartido es coincidencia, no un versionado emparejado."

Esto es un problema real y común: dos sistemas relacionados pueden usar la misma palabra ("v2") para cosas completamente distintas y no relacionadas, generando confusión para cualquiera que lea el código o la documentación después. Vale la pena que lo tengas presente como lección de nomenclatura: **cuando versionás algo, el número de versión no debería sugerir una relación con otro versionado cercano si no la hay realmente** — o, si el choque de nombres es inevitable, hay que documentarlo explícitamente como hicieron acá.

---

## 8. Cómo se ve esto en términos prácticos, para tu propio trabajo

Pensalo en el contexto de tu propio stack (Next.js + Supabase + Reparto con backend Hono):

- Si tu app cliente (PWA) y tu backend Hono se despliegan por separado (uno en Vercel, otro self-hosted), tenés exactamente el mismo problema de fondo: el cliente puede quedar corriendo una versión vieja mientras el backend ya cambió su forma de responder.
- Una estrategia mínima viable de esto sin todo el aparato de `gentle-pi`: versionar tus endpoints (`/api/v1/pedidos`, `/api/v2/pedidos`), o al menos versionar el *schema* de tus respuestas con algo como Zod, y tener tests que verifiquen que el schema que el backend produce y el que el cliente espera siguen siendo el mismo — eso ya es contract testing, aunque sea liviano.

---

## 9. Preguntas para seguir pensando

1. En un sistema con un solo repo (monorepo) donde frontend y backend se despliegan juntos, ¿tiene sentido igual invertir en contract testing formal, o el costo/beneficio cambia?
2. ¿Cómo manejarías la migración de v1 a v2 si no pudieras controlar cuándo se actualiza cada consumidor (por ejemplo, una API pública con miles de integraciones externas, no un plugin que vos mismo controlás)?
3. `gentle-pi` decidió no mantener un "dual-lane" permanente entre v1 y v2. ¿Cuándo tendría sentido lo contrario — mantener soporte a una versión vieja de forma indefinida?
4. Pensando en Reparto: si mañana necesitaras cambiar la forma en que el backend Hono le manda los pedidos al cliente PWA, ¿qué estrategia de las tres (big bang, compatibilidad perpetua, negociación explícita) tendría más sentido dado que probablemente controlás ambos lados del deploy?

---

## 10. Lectura de referencia

- *Building Microservices* (Sam Newman) — capítulo sobre versionado de APIs entre servicios independientes.
- Documentación de **Pact** (herramienta de contract testing) — para ver una implementación concreta de esta práctica.
- JSON Schema spec (`json-schema.org`) — el formato de contrato más común para APIs basadas en JSON.
- El repo mismo: [`Gentleman-Programming/gentle-pi`](https://github.com/Gentleman-Programming/gentle-pi), secciones "Bounded review transactions" y el package contents con `lib/review-integration-v2.ts`.
