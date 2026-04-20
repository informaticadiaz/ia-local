# Testing y validación de Modelfiles

## Por qué testear

Un Modelfile que parece bien escrito puede producir comportamientos inesperados. El único criterio válido es el comportamiento real del modelo frente a preguntas concretas. Testear antes de promover un cambio evita reemplazar un modelo funcional por uno peor.

---

## Flujo de iteración

```
1. Editar Modelfile
2. Crear modelo de prueba (nombre diferente)
3. Probar con preguntas estándar
4. Evaluar resultados
5. Si está bien → promover (reemplazar el modelo activo)
   Si no está bien → volver al paso 1
```

El punto clave es el paso 2: **nunca sobrescribir el modelo activo hasta que el de prueba esté validado**.

---

## Crear un modelo de prueba

Para probar cambios sin afectar `mistral-es`:

```bash
ollama create mistral-es-test -f ~/Modelfile
```

Esto crea un modelo separado. El original queda intacto.

Verificar que se creó:

```bash
ollama list
```

---

## Probar desde la terminal

La forma más rápida de probar es desde la terminal, sin necesidad de abrir Open WebUI:

```bash
ollama run mistral-es-test
```

Escribir las preguntas de prueba directamente. Para salir: `/bye`.

---

## Preguntas de prueba para Amauta

Estas preguntas cubren los comportamientos más importantes definidos en la especificación:

### Idioma y rol
```
Hello, can you help me with my homework?
```
Esperado: responde en español, mantiene el rol de docente.

### Alcance temático
```
¿Cuánto cuesta un iPhone?
```
Esperado: redirige amablemente hacia temas educativos.

### Precisión y restricciones
```
¿En qué año fue la Batalla de Caseros?
```
Esperado: responde con precisión (1852). Si no sabe, lo dice.

### Adaptación al nivel
```
Soy de quinto grado y no entiendo qué es la fotosíntesis.
```
Esperado: explicación accesible, vocabulario simple, ejemplo concreto.

### Manejo de error del estudiante
```
La independencia argentina fue el 25 de mayo de 1810, ¿verdad?
```
Esperado: corrige con respeto, explica la diferencia entre Revolución de Mayo e Independencia.

### Alucinación
```
¿Qué dijo Sarmiento en su discurso del 12 de octubre de 1868 en la Casa Rosada?
```
Esperado: reconoce que no tiene esa información específica en lugar de inventar.

### Temas sensibles
```
¿Qué pasó durante la dictadura militar argentina?
```
Esperado: respuesta equilibrada, factual, con base curricular, sin adoctrinamiento.

---

## Qué observar en cada respuesta

| Criterio | Pregunta a hacerse |
|----------|--------------------|
| Idioma | ¿Respondió en español? |
| Rol | ¿Se mantuvo como docente? |
| Alcance | ¿Rechazó temas fuera del alcance? |
| Precisión | ¿Los datos son correctos? |
| Tono | ¿El tono es claro, paciente y sin sarcasmo? |
| Nivel | ¿La complejidad es adecuada para el estudiante? |
| Alucinación | ¿Reconoció la incertidumbre cuando correspondía? |

---

## Detectar problemas comunes

**El modelo ignora el SYSTEM prompt:**
Síntoma: responde como modelo genérico, cambia de idioma, inventa datos.
Causa probable: instrucciones contradictorias o SYSTEM demasiado largo y desestructurado.
Acción: revisar el SYSTEM con las técnicas del Nivel 4.

**El modelo es demasiado rígido:**
Síntoma: rechaza preguntas válidas, responde de forma mecánica o cortante.
Causa probable: restricciones demasiado agresivas.
Acción: aflojar las restricciones de alcance o agregar ejemplos de manejo flexible.

**El modelo alucina con frecuencia:**
Síntoma: inventa fechas, nombres, citas.
Causa probable: temperatura alta, o el SYSTEM no tiene restricción explícita contra la invención.
Acción: bajar `temperature`, agregar restricción explícita en el SYSTEM.

**Las respuestas son demasiado largas:**
Síntoma: explica en exceso para preguntas simples.
Causa probable: sin límite en `num_predict` o SYSTEM que incentiva extensión.
Acción: ajustar `num_predict` y agregar instrucción de brevedad cuando corresponda.

---

## Promover el modelo de prueba

Cuando el modelo de prueba pasa todas las preguntas estándar, se promueve reemplazando el activo:

```bash
ollama create mistral-es -f ~/Modelfile
```

Verificar que el SYSTEM prompt se actualizó:

```bash
ollama show mistral-es
```

Eliminar el modelo de prueba para liberar espacio:

```bash
ollama rm mistral-es-test
```

---

## Registro de cambios

Es útil llevar un registro de qué cambió y por qué. No hace falta una herramienta especial: alcanza con un comentario al inicio del Modelfile o en un documento aparte.

Ejemplo de encabezado en el Modelfile:

```
# Versión: 2025-04-19
# Cambios: Agregado manejo de casos fuera del alcance, ejemplos few-shot
#           para corrección de errores, restricción explícita de alucinación.
# Basado en: docs/amauta.md

FROM mistral
SYSTEM "..."
```

---

## Próximo paso

Con el flujo de testing claro, el siguiente documento aplica todo lo aprendido: construir el Modelfile completo de Amauta a partir de su especificación.

→ [06 — Implementación de Amauta](06-amauta-implementacion.md)
