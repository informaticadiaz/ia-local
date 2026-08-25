# Spec-Driven Development (SDD) / OpenSpec

*Notas de estudio a partir del análisis de `gentle-pi`*

---

## 1. El problema de fondo: el chat es un mal lugar para guardar decisiones

Cuando trabajás con un agente de IA en una sesión larga, es muy común que las decisiones importantes — "por qué elegimos SQLite en vez de Postgres", "por qué el pedido tiene este estado intermedio y no otro" — queden **enterradas en algún punto del historial de chat**, mezcladas con exploración, prueba y error, y mensajes que no importan más.

Esto tiene consecuencias concretas:

- Si la ventana de contexto se comprime o resetea (algo que pasa constantemente en sesiones largas), esa decisión **desaparece** — literalmente deja de existir para el agente.
- Un colaborador humano (o vos mismo dentro de tres semanas) no puede auditar por qué se tomó una decisión sin releer todo el chat de punta a punta.
- No hay una versión "canónica" de cómo es el sistema hoy — solo un historial conversacional desordenado del que hay que inferir el estado actual.

SDD ataca esto con una idea simple pero poderosa: **las decisiones de arquitectura no viven en el chat, viven en archivos versionados en disco**, con una estructura fija y predecible. El chat pasa a ser el medio para *producir* esos archivos, no el lugar donde vive la información.

---

## 2. El flujo completo, fase por fase

```
init → explore → proposal → spec → design ─┬→ tasks → apply → verify → sync → archive
                                              └───────────┘
```

| Fase | Qué produce | Pregunta que responde |
|---|---|---|
| `init` | Configuración base del proyecto (`openspec/config.yaml`) | "¿Está el proyecto listo para trabajar con SDD?" |
| `explore` | Notas de investigación del código existente | "¿Cómo funciona hoy el sistema en el área que voy a tocar?" |
| `proposal` | Propuesta de cambio, en lenguaje de negocio/producto | "¿Qué problema resuelvo y por qué vale la pena?" |
| `spec` | Especificación formal de comportamiento (requisitos + escenarios) | "¿Qué tiene que hacer el sistema, exactamente?" |
| `design` | Decisiones técnicas de arquitectura | "¿Cómo lo voy a construir, y por qué así?" |
| `tasks` | Plan de tareas concretas y ordenadas | "¿En qué orden hago el trabajo?" |
| `apply` | El código en sí, más evidencia de progreso | "¿Está hecho? ¿Qué se hizo exactamente?" |
| `verify` | Reporte de verificación (tests, criterios de aceptación) | "¿Funciona como se especificó?" |
| `sync` | Actualización de las specs canónicas | "¿Cómo queda el estado 'oficial' del sistema después de este cambio?" |
| `archive` | Movimiento del cambio a un folder inmutable de auditoría | "¿Dónde queda el rastro histórico de esto, para siempre?" |

Cada fase **escribe un artefacto en disco**. No hay una fase que solo "converse" — todas dejan un rastro persistente y revisable.

---

## 3. La estructura de archivos como modelo de datos

```
openspec/
├── specs/                              ← fuente de verdad ACEPTADA
│   └── {dominio}/spec.md
└── changes/
    ├── {cambio}/                       ← trabajo EN CURSO
    │   ├── proposal.md
    │   ├── specs/{dominio}/spec.md     ← delta propuesto, todavía no aplicado
    │   ├── design.md
    │   ├── tasks.md
    │   ├── apply-progress.md
    │   ├── verify-report.md
    │   └── sync-report.md
    └── archive/YYYY-MM-DD-{cambio}/    ← rastro histórico INMUTABLE
```

Esto es, en esencia, un **modelo de datos de tres capas** aplicado a la documentación de arquitectura:

1. **Estado canónico** (`specs/`): "así es el sistema hoy, oficialmente aceptado".
2. **Estado en tránsito** (`changes/{cambio}/`): "esto es lo que estamos proponiendo cambiar, todavía no es verdad".
3. **Historial inmutable** (`changes/archive/`): "esto es lo que pasó, con fecha, para siempre".

Es la misma lógica de separación que usarías en una base de datos entre una tabla de estado actual y una tabla de auditoría/eventos — pero aplicada a documentos de texto versionados con Git en vez de filas de una tabla.

---

## 4. `sync` es la operación más interesante de todo el flujo

Vale la pena detenerse acá porque es donde el diseño se pone más cuidadoso. El README distingue explícitamente entre requisitos `ADDED`, `MODIFIED` y `REMOVED`:

```markdown
## ADDED Requirements

## MODIFIED Requirements

## REMOVED Requirements
```

Y una regla específica: *"Los requisitos `MODIFIED` tienen que incluir el bloque completo del requisito, incluyendo escenarios todavía válidos, porque sync reemplaza el bloque canónico por nombre de requisito."*

Esto es exactamente el mismo problema que resolvés con un **diff estructurado vs. un diff de texto plano**. Si `sync` simplemente pegara texto nuevo arriba del viejo, terminarías con specs canónicas llenas de contradicciones acumuladas ("el pedido tiene 3 estados" en un párrafo, "el pedido tiene 5 estados" en otro, sin que quede claro cuál es la verdad actual). Al exigir que un requisito `MODIFIED` venga completo, `sync` puede hacer un **reemplazo atómico por identidad** (por nombre de requisito), garantizando que la spec canónica nunca queda en un estado intermedio inconsistente.

Esto es conceptualmente idéntico a cómo Git resuelve un merge: no concatena los cambios ciegamente, opera sobre unidades bien definidas (líneas, hunks) con reglas claras de reemplazo.

---

## 5. Por qué esto es "el problema del legacy code", pero al revés

Hay una idea interesante escondida acá. En ingeniería de software tradicional, uno de los problemas más caros es el **código legacy sin documentación**: el sistema hace cosas, pero nadie sabe bien por qué, porque las decisiones originales se perdieron con el tiempo (la persona que las tomó ya no está, el ticket original se cerró hace años, etc.).

SDD ataca este problema **preventivamente**, no reparándolo después. En vez de escribir código primero y documentar (tal vez, si hay tiempo) después, el orden se invierte: **el artefacto de decisión existe antes o junto con el código**, no como un anexo opcional. Esto convierte la documentación de arquitectura de "una tarea que se posterga indefinidamente" a "un subproducto obligatorio del propio flujo de trabajo".

Es la misma filosofía detrás de TDD (escribir el test antes que el código obliga a pensar el comportamiento esperado antes de implementar) pero aplicada a decisiones de diseño en vez de a comportamiento verificable por tests.

---

## 6. Modos de persistencia: `openspec`, `engram`, `both`

El repo menciona tres modos posibles para dónde vive esta información:

| Modo | Dónde vive la info | Cuándo tiene sentido |
|---|---|---|
| `openspec` | Archivos en disco, versionados con Git | Necesitás historial auditable, revisable en PRs, sobrevive independientemente de cualquier herramienta de IA |
| `engram` | Memoria persistente entre sesiones (un sistema de memoria propio) | Trabajo más fluido, sin la ceremonia de mantener archivos, pero sin evolución canónica de specs |
| `both` (híbrido) | Ambos a la vez | Cuando querés lo mejor de los dos: memoria rápida para continuidad de sesión, y archivos para auditoría/versión de verdad |

El punto que marca el repo es importante: *"Engram-only mode es distinto por diseño: Engram es memoria de trabajo y no mantiene una capa de merge de spec canónica."* Es decir, memoria persistente (tipo `engram` o, en tu caso, algo tipo Amauta con su propio sistema) resuelve el problema de "no perder contexto entre sesiones", pero **no resuelve el problema de tener una única fuente de verdad versionada y revisable por humanos**. Son problemas relacionados pero distintos, y vale la pena no confundirlos.

---

## 7. La conexión directa con tu proyecto Amauta

Vos ya tenías una intuición muy similar antes de que llegáramos a este repo — `project-manager-automata`, `loop-runner.sh`, `issue-inspector`, `loop-auditor` son, en esencia, piezas de un sistema que intenta resolver el mismo problema: que un loop autónomo de Claude Code no pierda de vista qué está haciendo y por qué, a través de múltiples iteraciones.

Algunas preguntas concretas para comparar tu arquitectura actual con este modelo de fases:

- **¿Tu loop tiene una fase explícita de "propuesta" separada de "ejecución"?** SDD fuerza a que la propuesta (`proposal.md`) exista como artefacto *antes* de que se toque una línea de código, con su propio momento de revisión. Si `project-manager-automata` va directo de "acá hay un issue" a "generar código", podrías estar saltando ese paso de reflexión explícita.
- **¿Existe una spec canónica separada del historial de cambios?** Es la diferencia entre `specs/` (estado actual) y `changes/archive/` (historial). Si Amauta solo tiene logs/historial de lo que se hizo, pero no un documento "así es el sistema hoy", eso es exactamente el gap que `sync` + `specs/` resuelve.
- **¿`loop-auditor` cumple el rol de la fase `verify`?** Si es así, vale la pena ver si produce un artefacto persistente (`verify-report.md` equivalente) o si su resultado solo vive en la ejecución del momento.

---

## 8. Por qué esto no es "burocracia por la burocracia" — el costo tiene que justificarse

Es tentador leer este flujo de 10 fases y pensar "esto es demasiada ceremonia para un cambio chico". El propio repo lo reconoce con su tabla de ruteo por riesgo (que vimos en el tema 1): SDD se activa **solo cuando el riesgo del cambio lo justifica**. Un typo o un fix de una línea no pasa por `explore → proposal → spec → design`.

La pregunta de diseño real no es "¿SDD sí o no?", sino: **¿en qué punto el costo de la ceremonia se paga solo con la reducción de riesgo de perder contexto o de generar un cambio mal entendido?** Eso depende del tamaño del cambio, de si hay más de una persona (o sesión) involucrada, y de qué tan caro es deshacer un error si se descubre tarde.

---

## 9. Preguntas para seguir pensando

1. Si tuvieras que aplicar SDD a Reparto para una feature grande (por ejemplo, el sistema de rutas/asignación de repartidores), ¿qué fase creés que más valor te daría a vos como desarrollador solo, sin equipo?
2. ¿Cómo se comporta este modelo cuando dos cambios (`changes/`) tocan el mismo dominio en paralelo? El repo no lo explicita del todo — ¿qué estrategia usarías vos para evitar que dos `sync` se pisen?
3. Pensando en Amauta: ¿vale la pena portar la separación `specs/` (canónico) vs `changes/` (en tránsito) a tu loop actual, aunque sea de forma simplificada?
4. ¿Qué tan distinto sería este flujo si en vez de un agente de IA fuera un equipo humano? ¿Cuánto de esto es "buenas prácticas de ingeniería de software en general" y cuánto es específico a mitigar las debilidades particulares de un LLM (pérdida de contexto, alucinación)?

---

## 10. Lectura de referencia

- El proyecto **OpenSpec** en sí (referenciado por `gentle-pi` como el modelo de artefactos que implementa sin depender del CLI externo).
- *Architecture Decision Records (ADR)* — Michael Nygard — práctica clásica de "documentar decisiones como artefactos versionados", el ancestro directo conceptual de esto.
- *Documentation as Code* — la idea general de tratar la documentación con las mismas herramientas y disciplina que el código (versionado, revisión, CI).
- El repo mismo: [`Gentleman-Programming/gentle-pi`](https://github.com/Gentleman-Programming/gentle-pi), sección "SDD/OpenSpec flow" y "OpenSpec artifact model".
