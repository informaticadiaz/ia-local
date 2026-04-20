# Cómo funciona un LLM

## La idea central: predecir la siguiente palabra

Un modelo de lenguaje grande (LLM) hace una sola cosa fundamental: dado un texto, predice cuál es el token más probable que viene a continuación. Un token es aproximadamente una palabra o parte de una palabra.

Cuando escribís "La capital de Argentina es", el modelo calcula probabilidades para todos los tokens posibles y elige uno. Con alta probabilidad elige "Buenos", luego "Aires", y así construye la respuesta de a un token por vez.

Eso es todo. No "entiende" en el sentido humano. No tiene memoria entre conversaciones. No busca información en internet. Predice texto basándose en patrones aprendidos durante el entrenamiento.

---

## Qué es el contexto

El contexto es todo el texto que el modelo tiene disponible en un momento dado: el SYSTEM prompt, la conversación completa hasta ese punto, y el mensaje actual.

El modelo no recuerda conversaciones anteriores. Cada vez que abrís un chat nuevo, empieza desde cero. Lo que parece "memoria" dentro de una misma conversación es simplemente que todo el historial está incluido en el contexto.

El contexto tiene un límite de tamaño medido en tokens. El modelo `mistral-es` está configurado con un contexto de 32.768 tokens, lo que equivale a aproximadamente 25.000 palabras. Una vez que la conversación supera ese límite, los mensajes más viejos se descartan.

---

## Por qué funciona el SYSTEM prompt

Durante el entrenamiento, el modelo aprendió a seguir instrucciones. Se entrenó con millones de ejemplos donde un texto indicaba un rol o tarea y el modelo debía responder de acuerdo a eso.

El SYSTEM prompt aprovecha ese aprendizaje: es una instrucción que se coloca al inicio del contexto, antes que cualquier mensaje del usuario. El modelo lo interpreta como "estas son las reglas de esta conversación".

No es un mecanismo técnico especial. Es simplemente texto que el modelo aprendió a respetar durante el entrenamiento.

---

## La diferencia entre SYSTEM, usuario y asistente

En una conversación con un LLM hay tres tipos de mensajes:

| Rol | Quién lo escribe | Cuándo |
|-----|-----------------|--------|
| `system` | El desarrollador o configurador | Una sola vez, al inicio |
| `user` | La persona que usa el modelo | En cada turno de la conversación |
| `assistant` | El propio modelo | En cada respuesta |

El SYSTEM nunca aparece en la interfaz del usuario. Es una capa de configuración invisible que moldea cómo el modelo responde a todo lo demás.

En el caso de este proyecto, el Modelfile define el SYSTEM. Open WebUI lo envía automáticamente al modelo en cada conversación.

---

## Limitaciones reales

Conocer estas limitaciones evita frustraciones y ayuda a diseñar mejor el comportamiento del modelo.

**El modelo no siempre sigue las instrucciones.** El SYSTEM prompt aumenta la probabilidad de un comportamiento, pero no lo garantiza. Si las instrucciones son ambiguas, largas o contradictorias, el modelo puede ignorar partes de ellas.

**El modelo puede inventar información.** Este fenómeno se llama alucinación. El modelo genera texto plausible aunque no sea verdadero. No tiene acceso a bases de datos ni puede verificar lo que dice.

**El rendimiento depende del tamaño del modelo.** Un modelo de 7B parámetros como Mistral sigue instrucciones razonablemente bien, pero con menos precisión que modelos más grandes. Cuanto más específica y larga es la instrucción, más probable es que un modelo pequeño la cumpla de forma parcial.

**El contexto largo degrada la atención.** A medida que la conversación crece, el modelo le presta menos atención a lo que está lejos (incluyendo el SYSTEM prompt). En conversaciones muy largas el comportamiento puede desviarse gradualmente.

**El modelo no aprende durante la conversación.** Responder correctamente en un turno no mejora sus capacidades para el siguiente. Cada respuesta es independiente.

---

## Lo que esto implica para configurar el comportamiento

- Las instrucciones deben ser claras, concretas y sin contradicciones
- Menos instrucciones bien escritas funcionan mejor que muchas instrucciones vagas
- El modelo necesita repetición de conceptos clave si la conversación es larga
- Hay que probar y validar el comportamiento, no asumir que las instrucciones se cumplen
- Los parámetros de inferencia (temperatura, etc.) afectan cómo el modelo elige entre opciones — pero no qué opciones tiene disponibles

---

## Próximo paso

Con esta base clara, el siguiente documento explica qué es el SYSTEM prompt específicamente, cómo funciona dentro de un Modelfile y qué puede y no puede lograr.

→ [02 — El SYSTEM prompt](02-system-prompt.md)
