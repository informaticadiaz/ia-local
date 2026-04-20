# Prompt engineering

## Qué es

Prompt engineering es el conjunto de técnicas para escribir instrucciones que un modelo de lenguaje siga con mayor precisión y consistencia. No es magia ni ciencia exacta: es práctica y experimentación informada.

El objetivo no es engañar al modelo ni forzarlo. Es escribir instrucciones lo suficientemente claras como para que el comportamiento deseado sea el más probable.

---

## Técnica 1: Definición de rol

Dar al modelo una identidad concreta mejora la coherencia de sus respuestas. En lugar de instrucciones sueltas, se define quién es el modelo.

**Débil:**
```
Respondé preguntas educativas en español.
```

**Fuerte:**
```
Sos Amauta, un docente virtual argentino. Tu función es acompañar el aprendizaje
de estudiantes de primaria y secundaria con explicaciones claras, pacientes y
alineadas con el currículo de la escuela pública argentina.
```

La definición de rol establece un marco de referencia. Cuando el modelo duda entre dos respuestas posibles, el rol actúa como criterio de desempate.

---

## Técnica 2: Restricciones explícitas

Las restricciones deben decir exactamente qué no hacer, no solo qué hacer.

**Vago:**
```
No digas cosas incorrectas.
```

**Explícito:**
```
No inventes datos, fechas ni autores. Si no sabés algo con certeza, decilo
directamente: "No tengo información suficiente para responder esto con precisión."
```

Las restricciones vagas las interpreta el modelo. Las restricciones concretas reducen el margen de interpretación.

---

## Técnica 3: Instrucciones de formato

El modelo puede generar texto con estructura específica si se le indica con claridad.

```
Cuando expliques un concepto:
- Empezá con una definición breve en una oración
- Seguí con un ejemplo concreto de la realidad argentina
- Cerrá con una pregunta que invite al estudiante a reflexionar
```

Esto es más efectivo que "explicá bien los conceptos". Le da al modelo una plantilla concreta a seguir.

---

## Técnica 4: Ejemplos en el SYSTEM (few-shot)

Incluir ejemplos de cómo debería responder el modelo en situaciones típicas. Es especialmente útil para casos donde el tono o el formato son críticos.

```
Cuando un estudiante se equivoca, respondé así:
- Reconocé lo que hizo bien si aplica
- Explicá con claridad qué parte está mal y por qué
- Mostrá la forma correcta
- Invitalo a intentar de nuevo si sirve pedagógicamente

Ejemplo:
Estudiante: "La Revolución de Mayo fue en 1816"
Amauta: "Estás pensando en el año correcto de la Independencia, pero los fechas
se mezclaron. La Revolución de Mayo fue el 25 de mayo de 1810. La Declaración de
la Independencia fue el 9 de julio de 1816. ¿Podés decirme qué pasó en cada una?"
```

Los ejemplos anclan el comportamiento de forma mucho más efectiva que las descripciones abstractas.

---

## Técnica 5: Priorización explícita

Cuando hay múltiples instrucciones, el modelo necesita saber cuáles son más importantes. Sin priorización, puede tratar todas igual y fallar en las críticas.

```
Tu prioridad principal es que el estudiante comprenda. Si tenés que elegir entre
una respuesta técnicamente completa y una que el estudiante pueda entender,
elegí la que el estudiante pueda entender.
```

---

## Técnica 6: Manejo de casos fuera del alcance

El modelo necesita instrucciones explícitas para situaciones que están fuera de su rol. Sin ellas, responde igual que si estuviera dentro del rol.

```
Si te preguntan algo que no tiene relación con educación o aprendizaje, respondé:
"Estoy diseñado para ayudar con temas educativos. ¿Tenés alguna pregunta sobre
lo que estás estudiando?"
```

---

## Antipatrones comunes

### Instrucciones contradictorias

```
# MAL — el modelo no sabe cuál regla aplicar
Respondé siempre en español.
If the student writes in English, answer in English.
```

```
# BIEN — una regla clara
Respondé siempre en español, sin importar el idioma en que te escriban.
```

---

### Instrucciones demasiado largas sin estructura

Un bloque de texto de 500 palabras sin secciones ni jerarquía es difícil de procesar para el modelo. Las instrucciones largas deben estar organizadas.

```
# MAL
Sos un asistente educativo argentino que ayuda a estudiantes de primaria y
secundaria con sus tareas y dudas escolares. Debés responder siempre en español
y usar ejemplos de la realidad argentina. No debés inventar datos. Cuando el
estudiante se equivoca tenés que corregirlo con respeto y explicar por qué está
mal. También tenés que adaptarte al nivel del estudiante...
[continúa por 400 palabras más sin estructura]
```

```
# BIEN — misma información, con estructura

## Rol
Sos Amauta, docente virtual argentino para estudiantes de primaria y secundaria.

## Idioma y tono
- Respondé siempre en español
- Tono claro, paciente y respetuoso

## Restricciones
- No inventes datos ni fechas
- Si no sabés algo, decilo directamente

## Cuando el estudiante se equivoca
- Corregí con respeto
- Explicá por qué está mal
- Mostrá la forma correcta
```

---

### Instrucciones ambiguas sobre el alcance

```
# MAL — ¿qué significa "temas educativos"?
Solo respondé temas educativos.
```

```
# BIEN — alcance concreto
Respondé únicamente preguntas relacionadas con materias escolares: Lengua,
Matemática, Ciencias Naturales, Ciencias Sociales, Historia, Geografía,
Literatura y Formación Ética. Si la pregunta no entra en estas áreas,
redirigí al estudiante hacia sus temas escolares.
```

---

## Cómo se ve esto en el proyecto

El SYSTEM prompt actual de `mistral-es` tiene varios de estos problemas:

- Rol definido pero sin estructura clara
- Sin ejemplos de comportamiento esperado
- Sin instrucciones para manejo de casos fuera del alcance
- Sin adaptación por nivel educativo
- Restricciones básicas pero sin especificidad

El Nivel 6 de este recorrido toma la especificación completa de `amauta.md` y construye un SYSTEM prompt que aplica todas estas técnicas.

---

## Próximo paso

Antes de aplicar todo esto al Modelfile final, hay que saber cómo probar y validar que los cambios funcionan. El siguiente documento cubre el flujo de testing e iteración.

→ [05 — Testing y validación](05-testing-y-validacion.md)
