# El SYSTEM prompt

## Qué es

El SYSTEM prompt es una instrucción que se entrega al modelo antes de cualquier mensaje del usuario. Define el rol, el tono, el alcance y las reglas que el modelo debe seguir durante toda la conversación.

A diferencia de un mensaje en el chat, el SYSTEM prompt es invisible para el usuario. Se envía automáticamente al inicio de cada conversación.

---

## Cómo se define en un Modelfile

En Ollama, el SYSTEM prompt se establece con la directiva `SYSTEM` dentro del Modelfile:

```
FROM mistral
SYSTEM "Instrucción que define el comportamiento del modelo."
```

Todo lo que está entre comillas es el SYSTEM prompt. Se aplica a cada conversación que use ese modelo.

Para ver el SYSTEM prompt de un modelo instalado:

```bash
ollama show mistral-es
```

---

## Cómo lo usa el modelo

Cuando se inicia una conversación, el contexto que recibe el modelo tiene esta estructura:

```
[SYSTEM]   "Sos un asistente educativo argentino..."
[USER]     "¿Quién fue Manuel Belgrano?"
[ASSISTANT] ...
```

El modelo procesa todo esto junto. El SYSTEM prompt establece el marco dentro del cual interpreta y responde al mensaje del usuario.

No es un filtro que bloquea respuestas. Es texto que el modelo tiene en cuenta al calcular qué respuesta es más probable. Un SYSTEM prompt bien escrito hace que las respuestas deseadas sean más probables. Uno mal escrito o ausente deja al modelo con sus comportamientos por defecto.

---

## Qué puede hacer un SYSTEM prompt

- Definir un rol: "Sos un profesor de historia argentina"
- Establecer el idioma: "Respondé siempre en español"
- Limitar el alcance: "Solo respondé preguntas sobre matemática escolar"
- Definir el tono: "Usá lenguaje claro y accesible para estudiantes de secundaria"
- Dar instrucciones de formato: "Respondé con listas cuando corresponda"
- Establecer restricciones: "No inventes datos. Si no sabés algo, decilo"

---

## Qué NO puede hacer un SYSTEM prompt

- Garantizar que el modelo nunca se equivoque
- Eliminar completamente las alucinaciones
- Darle al modelo conocimiento que no tiene
- Obligar al modelo a seguir instrucciones que contradicen fuertemente su entrenamiento
- Mantener el comportamiento perfectamente estable en conversaciones muy largas

Un SYSTEM prompt es una guía fuerte, no una programación determinista.

---

## El SYSTEM prompt actual de este proyecto

El modelo `mistral-es` tiene este SYSTEM prompt:

```
Sos un asistente educativo argentino. Responde en español, saludos breves e
informacion conciza. Te encargas de responder dudas de alumnos con base a temas
educativos argentinos, no tenes que responder o hablar de ningun tema en particular
salvo que te lo pregunten, priorizando informacion historica, cultural y educativa
de Argentina. No inventes datos. Si hay ambiguedad eleegi lo mas relevante para
Argentina. Explica conceptos educativos con precision y sin desviarte.
```

Es un SYSTEM prompt funcional pero incompleto. Cubre el idioma, el rol general y algunas restricciones básicas. Sin embargo, comparado con la especificación completa de `amauta.md`, le faltan:

- Definición del público (primaria, secundaria, docentes)
- Adaptación por nivel educativo
- Tono y estilo de comunicación detallado
- Manejo de errores del estudiante
- Marco curricular (NAP, diseños curriculares)
- Límites de seguridad

---

## Buenas prácticas

**Ser específico.** "Usá ejemplos concretos de la vida cotidiana argentina" es mejor que "sé claro".

**Evitar contradicciones.** Si el SYSTEM dice "respondé en español" y también "respond in English when asked", el modelo va a tener comportamiento inconsistente.

**Priorizar las instrucciones más importantes.** El modelo presta más atención al inicio del SYSTEM prompt. Las reglas críticas van primero.

**No sobrecargar.** Un SYSTEM prompt de 20 instrucciones detalladas va a ser seguido parcialmente. Mejor 5 instrucciones claras y prioritarias.

**Probar después de cada cambio.** El único criterio válido es el comportamiento real del modelo, no lo que parece razonable en papel.

---

## Próximo paso

El SYSTEM prompt controla qué hace el modelo. Los parámetros de inferencia controlan cómo lo hace: qué tan creativo es, qué tan largo responde, si tiende a repetirse. El siguiente documento cubre la directiva `PARAMETER` del Modelfile.

→ [03 — Parámetros del Modelfile](03-parametros-modelfile.md)
