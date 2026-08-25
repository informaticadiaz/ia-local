# El Patrón Harness/Middleware sobre Agentes de IA

*Notas de estudio a partir del análisis de `gentle-pi`*

---

## 1. El problema de fondo

Un LLM operando como agente (leer código, escribir código, ejecutar comandos, decidir próximos pasos) es un componente **potente pero no determinístico**. Eso lo hace excelente para tareas abiertas y pésimo como única fuente de control de proceso, porque:

- No tiene memoria persistente confiable entre pasos (todo vive en una ventana de contexto que se puede comprimir o perder).
- Puede "narrar" que hizo algo sin que sea estrictamente cierto (alucinación de estado).
- No tiene incentivo interno para frenar y pedir revisión — tiende a seguir generando hasta terminar la tarea que interpretó.
- Su comportamiento varía entre ejecuciones aunque el input sea similar.

Esto es un problema conocido en ingeniería de software con componentes no confiables: **no podés poner la lógica de negocio crítica adentro del componente que no controlás completamente.** La solución clásica es envolverlo.

---

## 2. La analogía con Middleware / Interceptores

En arquitecturas de software tradicionales, el patrón **Middleware** (o **Interceptor**) resuelve un problema estructuralmente idéntico: tenés una petición que atraviesa un sistema, y necesitás insertar comportamiento transversal (auth, logging, validación, rate limiting) **sin modificar el componente que procesa la petición**.

```
Request → [MW: Auth] → [MW: Logging] → [MW: Validation] → Handler → Response
```

Cada middleware puede:
- Inspeccionar la petición antes de que llegue al handler.
- Bloquearla (fail closed) si no cumple una condición.
- Modificarla o enriquecerla.
- Registrar evidencia de que pasó por ahí (auditoría).

`gentle-pi` aplica exactamente esta estructura, pero la "petición" no es un HTTP request — es una **acción propuesta por el agente** (escribir un archivo, hacer commit, abrir un PR, mergear).

```
Acción propuesta por Pi → [Check: ¿es chica y local?] → [Check: ¿toca 2+ archivos no triviales?]
  → [Check: ¿requiere SDD?] → [Check: revisión de riesgo] → [Check: evidencia de Git válida] → Ejecución real
```

### Tabla comparativa

| Middleware tradicional | Harness sobre agente |
|---|---|
| Intercepta requests HTTP | Intercepta acciones del agente (edición de archivo, commit, push) |
| Verifica JWT / sesión | Verifica que el "candidato" de cambios esté congelado y sea el correcto |
| Rate limiting por IP | Presupuesto de líneas de corrección, budget de revisión |
| Logging de auditoría | Receipts atados por hash al árbol de Git exacto |
| Rechaza con 401/403 | "Fail closed": bloquea con un envelope tipado de error |
| No modifica el handler de negocio | No modifica los pesos ni el comportamiento del LLM |

La diferencia clave es que en el middleware tradicional el pipeline es **completamente determinístico** en ambos extremos (request y handler). Acá, un extremo (el agente) es no determinístico, y el harness existe *precisamente* para compensar eso.

---

## 3. Por qué "no tocar el modelo" es una decisión de diseño, no una limitación

`gentle-pi` podría, en teoría, intentar resolver esto con mejores prompts ("por favor sé cuidadoso, hacé TDD, no generes diffs gigantes"). Es la solución ingenua y es la que la mayoría de la gente prueba primero. Falla por una razón estructural:

> **Un prompt es una sugerencia probabilística, no una garantía.** No podés usar texto en lenguaje natural como mecanismo de control cuando necesitás una invariante dura (ej: "nunca se hace push sin revisión aprobada").

Esto conecta con un principio general de sistemas confiables: **las garantías de seguridad y proceso no pueden vivir en la misma capa que decide qué hacer**. Tienen que vivir en una capa separada que:

1. No puede ser persuadida ni "jailbreakeada" por lenguaje natural, porque no interpreta lenguaje natural — interpreta estado verificable (hashes de Git, existencia de archivos, resultados de comandos).
2. Es determinística: mismos inputs → mismos outputs, siempre.
3. Falla de forma segura (fail closed) ante cualquier ambigüedad, en vez de asumir que "probablemente está bien".

Esto es exactamente el mismo argumento que se usa para no poner lógica de autorización en el frontend de una app web: el frontend puede ser persuadido/manipulado, así que la autoridad real vive en el backend.

---

## 4. Pipeline como maquinaria auditable

La sección del README sobre routing lo deja explícito:

| Forma del pedido | Harness |
|---|---|
| Cambio chico, claro, local | Trabajo directo inline |
| Área desconocida o exploración pesada | Delegación a subagente enfocado |
| Cambio grande, ambiguo, arquitectónico | Flujo SDD/OpenSpec completo |

Esto es un **router de complejidad**: no todo pasa por el mismo camino. Es análogo a un *circuit breaker* o a un *load balancer con reglas de ruteo* — la decisión de "qué tan pesado es el control que aplico" depende de una clasificación previa del riesgo de la tarea.

Para estudiar esto en términos de ingeniería de software, tres ideas transferibles:

- **Principio de menor ceremonia posible**: el harness no aplica el proceso más pesado a todo (eso sería SDD para un typo). Aplica exactamente el nivel de control proporcional al riesgo detectado.
- **Idempotencia y reintentos seguros**: las validaciones se pueden reintentar sin duplicar efectos (mencionado en el repo como CAS — content-addressable storage).
- **Fail closed por defecto**: ante cualquier estado ambiguo, corrupto o no reconocido, el sistema bloquea en vez de continuar optimistamente. Esto es lo opuesto al comportamiento típico de un LLM suelto, que tiende a "seguir adelante e interpretar" cuando algo no está claro.

---

## 5. Dónde encaja esto en "Agentic Engineering"

Esta es un área nueva de ingeniería de software que se está formando ahora mismo (2025-2026), y vale la pena que la ubiques dentro del mapa general:

```
                    ┌─────────────────────────────┐
                    │   Prompt Engineering         │  → cómo pedirle bien al modelo
                    └─────────────────────────────┘
                                  │
                    ┌─────────────────────────────┐
                    │   Context Engineering        │  → qué información meterle en la ventana
                    └─────────────────────────────┘
                                  │
                    ┌─────────────────────────────┐
                    │   Harness / Agentic          │  → cómo estructurar el PROCESO
                    │   Engineering                │     alrededor del modelo:
                    │                               │     control, delegación, auditoría,
                    │                               │     recuperación de errores
                    └─────────────────────────────┘
```

El harness engineering asume que el modelo ya es lo suficientemente bueno como componente de razonamiento, y se enfoca en el problema **de sistemas**: orquestación, memoria persistente, delegación a subagentes, control de alcance, evidencia verificable, recuperación ante fallos. Es el mismo salto conceptual que hubo en software tradicional entre "escribir una función que funciona" y "diseñar el sistema que la rodea para que sea confiable en producción" (logging, retries, circuit breakers, observabilidad).

---

## 6. Preguntas para seguir pensando (útiles para un trabajo o práctica)

1. ¿Qué pasaría si el harness mismo tuviera un bug — quién audita al auditor? (Pista: el repo lo reconoce explícitamente como "non-goal" ante un proceso malicioso del mismo usuario con acceso de escritura.)
2. ¿Este patrón es generalizable a *cualquier* agente (Claude Code, Cursor, etc.) o depende de que la plataforma exponga hooks/extensiones suficientes?
3. ¿Cómo se compara el costo de "ceremonia" (SDD completo) contra la velocidad de iteración? ¿Dónde está el punto óptimo?
4. En tu propio proyecto Amauta, ¿el loop autónomo tiene algún equivalente a este "trust boundary" entre lo que el agente afirma y lo que el sistema puede verificar objetivamente?

---

## 7. Lectura de referencia

- Patrón Middleware / Chain of Responsibility (Gang of Four) — la base clásica de este diseño.
- Circuit Breaker pattern (Michael Nygard, *Release It!*) — para el routing por riesgo.
- El repo mismo: [`Gentleman-Programming/gentle-pi`](https://github.com/Gentleman-Programming/gentle-pi), sección "Trust boundary" y "Bounded review transactions".
