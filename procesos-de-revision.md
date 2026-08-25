# Máquina de Estados para el Proceso de Revisión

*Notas de estudio a partir del análisis de `gentle-pi`*

---

## 1. Por qué modelar esto como state machine y no como código imperativo

La forma "ingenua" de programar un flujo de revisión sería algo así:

```javascript
// MAL: lógica de flujo dispersa en ifs anidados
function processReview(candidate) {
  const findings = runReview(candidate);
  if (findings.some(f => f.severity === 'severe')) {
    const fix = applyCorrection(candidate);
    if (testsPass(fix)) {
      if (!alreadyCorrectedOnce) {
        // ¿y si esto se llama de nuevo? ¿y si alreadyCorrectedOnce está mal seteado?
        approve(fix);
      }
    } else {
      // ¿escalamos? ¿reintentamos? depende de variables sueltas
    }
  } else {
    approve(candidate);
  }
}
```

El problema no es que esto no *funcione* — es que **los estados válidos del sistema quedan implícitos**, dispersos en combinaciones de variables booleanas y banderas. Nadie puede mirar el código y responder con certeza: "¿cuáles son TODOS los estados posibles en los que puede estar una revisión ahora mismo?"

Una **máquina de estados finita (FSM)** invierte el enfoque: primero definís explícitamente el conjunto cerrado de estados posibles, y después definís qué transiciones están permitidas entre ellos. Todo lo que no sea una transición explícitamente permitida, **no existe**.

---

## 2. La FSM de `gentle-pi`

```
        ┌───────────┐
        │ reviewing │  ← estado inicial, tras START congela el candidato
        └─────┬─────┘
              │ se corren los lentes seleccionados
              ▼
      ┌───────────────┐
      │ ¿bloqueador    │
      │ severo real?   │
      └───┬───────┬────┘
          │no     │sí
          ▼       ▼
    ┌──────────┐ ┌──────────────────────┐
    │ approved │ │ correction_required   │
    └──────────┘ └──────────┬────────────┘
                             │ se aplica UN fix acotado
                             ▼
                      ┌─────────────┐
                      │ validating  │
                      └──────┬──────┘
                     pasa │      │ falla / malformado / fuera de scope
                          ▼      ▼
                   ┌──────────┐ ┌────────────┐
                   │ approved │ │ escalated  │
                   └──────────┘ └────────────┘
```

Cinco estados. Nada más. `reviewing`, `correction_required`, `validating`, `approved`, `escalated`. El sistema **termina solo en `approved` o `escalated`** — no hay ningún camino que deje una revisión "flotando" en un estado ambiguo.

---

## 3. Los tres mecanismos de diseño que la hacen "dura"

### 3.1 Un solo ciclo de corrección — no hay bucle de reintento infinito

Esta es la decisión de diseño más importante del flujo, y vale la pena que la notes bien: **no hay una arista que vuelva de `validating` a `correction_required`.**

```
validating --falla--> escalated   (NO vuelve a correction_required)
```

¿Por qué importa esto? Porque un bucle `intentar → fallar → reintentar → fallar → reintentar...` es exactamente el patrón que hace que un sistema con un componente no determinístico (el LLM corrigiendo código) se vuelva **impredecible en tiempo y en resultado**. Sin este límite, nada te garantiza que el proceso converja — podría, en el peor caso, iterar indefinidamente sin nunca llegar a un estado terminal.

Al eliminar la arista de retorno, la FSM garantiza matemáticamente que el proceso **termina en como máximo 2 pasos desde que se detecta un problema**: un intento de corrección, una validación. Si eso no alcanza, el sistema deja de intentar solo y escala a un humano (`escalated`). Esto es una garantía de **terminación acotada**, una propiedad que en sistemas concurrentes o con reintentos automáticos hay que diseñar explícitamente — no viene gratis.

### 3.2 Presupuesto cuantitativo como invariante de la transición

```
correction_budget = min(200, ceil(original_changed_lines / 2))
```

Esto no es solo una regla de negocio — es una **guarda (guard condition)** de la transición `correction_required → validating`. En términos formales de FSM, una guarda es una condición booleana que debe cumplirse para que la transición sea válida; si no se cumple, la transición no ocurre (en este caso, la corrección se rechaza y probablemente se va directo a `escalated`).

Fijate el diseño de la fórmula: crece con el tamaño del cambio original, pero tiene un techo fijo (200 líneas). Esto evita dos fallas simétricas:
- Un cambio chico (10 líneas) no puede "corregirse" con una reescritura de 500 líneas — sería sospechoso y probablemente indica que la corrección se salió de scope.
- Un cambio enorme no obtiene un presupuesto de corrección ilimitado solo por ser grande — el techo de 200 lo previene.

### 3.3 El validador tiene poderes deliberadamente limitados

> "The validator cannot change claims, add findings, request fixes, launch actors, or request another attempt."

Esto es clave y fácil de pasar por alto: el estado `validating` **no es un mini-`reviewing`**. Es un chequeo estrecho y específico — solo confirma si la corrección puntual resolvió el problema puntual que la originó, contra los criterios ya congelados. No puede reabrir el análisis, no puede encontrar cosas nuevas, no puede pedir "una vuelta más". 

Esto es una aplicación del **principio de responsabilidad única** llevado al diseño de estados: cada estado tiene una función estrictamente acotada. Si el estado `validating` tuviera la misma potencia que `reviewing`, en la práctica tendrías un bucle disfrazado de dos estados distintos — la garantía de terminación del punto 3.1 se rompería en la práctica aunque el diagrama "parezca" acíclico.

---

## 4. Invariantes duros vs invariantes blandos

Vale la pena que distingas estos dos conceptos, porque es la idea central de por qué esta FSM es "dura":

- **Invariante blando**: una regla que el sistema *intenta* cumplir, pero que puede violarse en casos raros sin que el sistema se detenga (ej: "el LLM debería revisar bien" — no hay garantía).
- **Invariante duro**: una regla que el sistema **garantiza estructuralmente**, porque violarla requeriría una transición que simplemente no existe en el grafo de estados.

Ejemplos de invariantes duros en esta FSM:

| Invariante | Cómo se garantiza estructuralmente |
|---|---|
| Nunca hay más de un ciclo de corrección | No existe arista `validating → correction_required` |
| El proceso siempre termina | Todo camino converge a `approved` o `escalated`, sin ciclos |
| Un hallazgo "clean" también es exigente | `findings: []` sigue requiriendo `evidence: [...]` — no se puede aprobar sin registrar evidencia, ni siquiera cuando no hay problemas |
| El fix no puede exceder su presupuesto | Guarda cuantitativa en la transición, no una convención que el LLM podría ignorar |

La diferencia práctica: un invariante blando lo podés romper por accidente (un prompt mal escrito, un LLM que "decide" ignorar la instrucción). Un invariante duro solo se puede romper si hay un bug en la implementación de la FSM misma — no por un error de razonamiento del agente.

---

## 5. Fail-closed como propiedad de la FSM, no como excepción

Fijate que **ninguna transición en el diagrama tiene un estado de "seguir como si nada"**. Cualquier cosa que no encaje limpio en una transición conocida (evidencia faltante, corrección malformada, fuera de scope) empuja hacia `escalated`, nunca hacia `approved`.

Esto contrasta con el patrón típico de manejo de errores donde el "camino feliz" está bien definido y los errores se manejan con excepciones sueltas, `try/catch` dispersos, o simplemente se ignoran. Acá, **el estado de error (`escalated`) es un ciudadano de primera clase del diagrama**, tan formal como `approved`. Esto es lo que en diseño de FSMs se llama a veces "estado sumidero explícito" (*explicit sink state*): en vez de dejar que el sistema termine en un estado indefinido, definís un estado terminal específico para "esto necesita intervención humana" y todo camino problemático converge ahí.

---

## 6. Por qué esto es particularmente importante *con* un LLM en el loop

Una FSM con invariantes duros es una buena práctica en cualquier sistema, pero se vuelve casi **obligatoria** cuando uno de los actores que dispara transiciones es no determinístico:

- Si el LLM pudiera, aunque sea en teoría, generar una secuencia de acciones que produjera un bucle infinito de correcciones, el costo (tiempo, tokens, dinero) sería impredecible y potencialmente ilimitado.
- Si el LLM pudiera "convencer" al sistema de agregar una transición nueva no contemplada (ej. "dejame intentar una vez más"), el diseño entero de invariantes duros colapsaría a invariantes blandos.
- Al mantener el conjunto de estados y transiciones **fijo y cerrado**, el LLM solo puede operar *dentro* del espacio de comportamiento que el diseño permite — nunca puede expandir ese espacio desde adentro.

Esto conecta directo con el tema del trust boundary que charlamos antes: la FSM es, en cierto sentido, la **forma concreta y ejecutable** de ese trust boundary. No es solo "el LLM no decide" en abstracto — es "el LLM no tiene ninguna transición disponible que le permita decidir", porque esa transición simplemente no está en el grafo.

---

## 7. Preguntas para seguir pensando

1. ¿Cómo modelarías en una FSM un caso donde el "corrector" es un humano y no un LLM? ¿Cambiaría el límite de un solo ciclo de corrección, o seguiría siendo buena práctica?
2. La fórmula `min(200, ceil(original_changed_lines / 2))` es una guarda cuantitativa simple. ¿Qué otras guardas (no solo de líneas) tendrían sentido agregar a una transición como esta — por ejemplo, relacionadas a qué archivos toca la corrección?
3. Si tuvieras que dibujar la FSM completa de tu loop de Amauta (`project-manager-automata`), ¿cuáles son sus estados reales hoy? ¿Hay transiciones implícitas que deberían ser explícitas?
4. ¿Qué herramientas usarías para *verificar* que una implementación de código realmente respeta el diagrama de una FSM (y no tiene, por accidente, un camino de código que crea una transición no contemplada)?

---

## 8. Lectura de referencia

- Máquinas de estado finito (FSM) — teoría de autómatas, base formal de todo este diseño.
- *Statecharts: A Visual Formalism for Complex Systems* (David Harel) — extensión de FSMs para sistemas reactivos complejos, útil si el diagrama simple no alcanza.
- Patrón *State* (Gang of Four) — cómo implementar FSMs limpiamente en código orientado a objetos.
- El repo mismo: [`Gentleman-Programming/gentle-pi`](https://github.com/Gentleman-Programming/gentle-pi), sección "Bounded review transactions".
