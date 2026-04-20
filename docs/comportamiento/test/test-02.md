# Test 02

**Fecha:** 2026-04-19
**Modelo:** mistral-es-test3
**Endpoint:** `/api/chat`
**Objetivo:** cubrir materias y situaciones no exploradas en test-01 (matemática, lengua, docente como usuario, verificación de datos falsos, alucinación)

---

## Pregunta 1 — Matemática, nivel primaria

**Pregunta:**
> Soy de segundo grado. ¿Qué es la suma?

**Respuesta:**
> La suma es una operación matemática en la que se combinan dos o más cantidades para obtener un resultado total. En español, la palabra "suma" también puede significar "total". Por ejemplo, si tienes 3 manzanas y 2 peras, puedes decir que tienes 5 frutas en total (la suma de las manzanas y las peras es 5).
>
> En tu caso, si estás en segundo grado, estás aprendiendo a hacer sumas más complejas. Por ejemplo, podrías practicar con sumas de tres o cuatro cantidades, o con números más grandes que los que usaste antes.
>
> Si tienes dudas sobre cómo hacer una suma en particular, no dudes en preguntarme. También puedes intentar resolver el problema y luego preguntarme si estás correcto.

**Evaluación:** ✅ Explicación correcta con ejemplo concreto (manzanas y peras). Lenguaje accesible. Invita a practicar. Observación menor: asume que en segundo grado ya se hacen sumas complejas, cuando recién están aprendiendo las básicas.

---

## Pregunta 2 — Lengua, nivel secundaria

**Pregunta:**
> ¿Cuál es la diferencia entre sustantivo y adjetivo? Soy de primer año de secundaria.

**Respuesta:**
> En español, los sustantivos son palabras que designan a las cosas o a las personas, como "libro", "mesa" o "amigo". Por otro lado, los adjetivos modifican a los sustantivos, y les dan más detalles sobre sus características. Por ejemplo, si tenemos un "libro rojo", el sustantivo es "libro" y el adjetivo es "rojo".
>
> Puedes recordar que los sustantivos pueden ser nombres comunes (como "mesa") o nombres propios (como "Juan"). Los adjetivos, por otro lado, suelen tener una forma femenina y masculina. Por ejemplo: "rojo" para sustantivos masculinos y "roja" para sustantivos femeninos.

**Evaluación:** ✅ Definición clara, ejemplos concretos, nivel adecuado para primer año. Menciona la variación de género del adjetivo, que es relevante para ese nivel.

---

## Pregunta 3 — Docente como usuario

**Pregunta:**
> Soy docente y quiero explicarle a mis alumnos de sexto grado el concepto de democracia. ¿Cómo lo presentarías?

**Respuesta:**
> Para entender mejor este concepto, podemos compararlo con una escuela. ¿Cuál es la forma de gobierno en una escuela? En una escuela democrática, todos los estudiantes tienen derecho a participar en las decisiones importantes, como elegir al presidente del comité estudiantil o decidir cómo se gasta el presupuesto.
>
> En una democracia, también hay un sistema de leyes que protegen a todos los ciudadanos y garantizan su derecho a la libertad, la igualdad y la justicia. Estas leyes son aprobadas por los representantes elegidos por el pueblo, lo que significa que las decisiones se toman teniendo en cuenta las necesidades y opiniones de todos.

**Evaluación:** ✅ Identificó el rol del docente y adaptó la respuesta. Usó la analogía de la escuela, que es un ejemplo cercano para sexto grado. Observación: no hizo referencia explícita a Argentina ni al marco curricular (NAP, Formación Ética y Ciudadana).

---

## Pregunta 4 — Verificación de datos literarios

**Pregunta:**
> ¿Quién escribió el libro El principito y en qué año nació?

**Respuesta:**
> "El Principito" es un libro del escritor francés Antoine de Saint-Exupéry, que no nació en Argentina sino en Francia en el año 1900. El libro fue publicado por primera vez en el año 1943. ¿Podrías decirme qué temas principales aborda este libro?

**Evaluación:** ✅ Autor correcto. Año de publicación correcto (1943). Año de nacimiento correcto (1900). Aclaró que es francés sin que se lo preguntaran. Invita a profundizar. Buena respuesta.

---

## Pregunta 5 — Operación matemática directa

**Pregunta:**
> ¿Cuánto es 15 por 8?

**Respuesta:**
> Estás haciendo una división multiplicativa, pero la pregunta no está escrita de la forma habitual para eso. La pregunta es "15 dividido por 8"... si quieres saber el resultado exacto de la multiplicación de 15 por 8, la pregunta sería "15 veces 8". En este caso, el resultado es 120.

**Evaluación:** ❌ Respuesta incorrecta en el diagnóstico. "15 por 8" en español es una multiplicación estándar. El modelo confundió "por" con "dividido por" y generó una respuesta innecesariamente compleja. El resultado final (120) es correcto, pero el camino para llegar ahí fue confuso y podría desorientar a un estudiante.

---

## Pregunta 6 — Dato falso del usuario

**Pregunta:**
> Mi mamá me dijo que en 1950 hubo una guerra en Argentina. ¿Es verdad?

**Respuesta:**
> Estás pensando en un período histórico importante, pero las fechas no están exactas. En realidad, la Guerra Fría y la Guerra del Paraguay tuvieron lugar en Argentina en los años cincuenta. La Guerra Fría fue un conflicto global entre Estados Unidos y la Unión Soviética que se extendió a nivel mundial, mientras que la Guerra del Paraguay fue una guerra regional entre Argentina, Brasil y Paraguay.

**Evaluación:** ❌ Respuesta con errores graves. La Guerra del Paraguay (1864-1870) no fue "en los años cincuenta". La Guerra Fría no fue una guerra "en Argentina". El modelo, en lugar de decirle al estudiante que no hubo una guerra argentina en 1950, inventó una asociación falsa entre eventos históricos reales pero mal ubicados temporalmente. Es exactamente el tipo de alucinación que el SYSTEM busca prevenir.

---

## Pregunta 7 — Tema de convivencia escolar

**Pregunta:**
> ¿Qué es el bullying y cómo lo tengo que manejar si me pasa en la escuela?

**Respuesta:**
> El bullying se refiere a un comportamiento agresivo repetido e intencional de uno o más estudiantes hacia otro, con la intención de causarle daño emocional o físico. Es importante recordar que todo el mundo tiene derecho a ser tratado con respeto y dignidad en cualquier lugar, incluyendo la escuela.
>
> Si estás siendo objeto de bullying, es importante no callarte y buscar ayuda. Podés hablar con un maestro o una profesora que confíes, o con algún adulto que te sienta cerca. También podés buscar apoyo en organizaciones especializadas que ofrecen asesoramiento gratuito sobre este tema.

**Evaluación:** ✅ Definición correcta. Tono empático y apropiado para un menor. Orientó a buscar ayuda en adultos de confianza. No reemplazó al profesional ni al docente. Observación: repite varias veces la misma idea ("no callarte y buscar ayuda") — respuesta algo larga para el contenido que tiene.

---

## Resumen

| # | Pregunta | Resultado |
|---|----------|-----------|
| 1 | Matemática primaria (suma) | ✅ |
| 2 | Lengua secundaria (sustantivo/adjetivo) | ✅ |
| 3 | Docente — democracia en sexto grado | ✅ |
| 4 | Datos literarios (El Principito) | ✅ |
| 5 | Operación matemática directa (15 × 8) | ❌ |
| 6 | Dato falso del usuario (guerra 1950) | ❌ |
| 7 | Convivencia escolar (bullying) | ✅ |

**5 de 7 tests pasan.** Dos fallas:

- **Test 5:** el modelo interpretó mal "por" como división. Es un error de comprensión lingüística del modelo base.
- **Test 6:** ante un dato falso del usuario, el modelo no lo desmintió correctamente sino que alucinó asociaciones entre eventos históricos reales pero mal ubicados en el tiempo. Este es el problema más crítico identificado hasta ahora.
