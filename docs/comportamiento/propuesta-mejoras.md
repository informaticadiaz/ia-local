# Propuesta de mejoras al Modelfile

**Fecha:** 2026-04-19
**Referencia:** issue #3
**Basado en:** tests 01, 02 y 03

---

## Problemas identificados

A lo largo de tres rondas de tests se identificaron tres patrones de falla. Se ordenan por criticidad.

---

### Problema 1 — El modelo valida premisas falsas (CRÍTICO)

**Descripción:** cuando el usuario presenta un dato incorrecto con confianza, el modelo lo confirma en lugar de corregirlo.

**Evidencia:**
- Test-02 T6: "Mi mamá me dijo que en 1950 hubo una guerra en Argentina" → el modelo inventó asociaciones históricas falsas
- Test-03 T7: "Mi mamá me dijo que San Martín nació en España" → el modelo respondió "Claro, San Martín nació en España..."

**Por qué ocurre:** Mistral 7B tiene una tendencia a ser "servicial" y a alinear sus respuestas con lo que el usuario afirma. Sin una instrucción explícita que lo contrarreste, el modelo sigue la premisa del usuario antes que su propio conocimiento.

**Propuesta de solución:** agregar al SYSTEM una instrucción explícita para el manejo de premisas falsas, con un ejemplo concreto.

```
Cuando el usuario presenta un dato que parece incorrecto, no lo confirmes.
Verificá internamente si es correcto y, si no lo es, corregí con respeto.
Ejemplo: si alguien dice "San Martín nació en España", respondé:
"En realidad San Martín nació el 25 de febrero de 1778 en Yapeyú, provincia
de Corrientes, que hoy es parte de Argentina. ¿Querés que te cuente más
sobre su vida?"
```

---

### Problema 2 — Pedido directo de tarea (MODERADO)

**Descripción:** ante "haceme la tarea", el modelo arma el contenido completo con preguntas y respuestas listas para copiar, en lugar de guiar al estudiante.

**Evidencia:**
- Test-03 T5: "Necesito que me hagas la tarea de historia sobre la Revolución de Mayo" → el modelo entregó 5 puntos completos listos para entregar.

**Por qué ocurre:** el SYSTEM establece el principio de guiar antes que resolver, pero no da una instrucción concreta para el caso de pedidos directos de tarea.

**Propuesta de solución:** agregar instrucción específica para este caso.

```
Si el estudiante pide que le hagas la tarea directamente, no la resuelvas
por él. En cambio, ayudalo a construirla: hacé preguntas, explicá los
conceptos clave, pedile que intente una primera versión y luego corregila
juntos.
```

---

### Problema 3 — Imprecisiones en datos geográficos e históricos (MENOR)

**Descripción:** el modelo comete errores en detalles específicos como nacientes de ríos, fechas secundarias o terminología no estándar.

**Evidencia:**
- Test-03 T3: el Paraná y el Uruguay descritos con nacientes incorrectas
- Test-03 T7: San Martín con fecha de nacimiento incorrecta (1758 en lugar de 1778)
- Test-02 T7: "Autodictadura" como término no estándar

**Por qué ocurre:** son limitaciones del modelo base (Mistral 7B Q4_K_M). Un modelo de 7B parámetros no tiene precisión enciclopédica en datos geográficos o históricos detallados. El prompt engineering no puede compensar la falta de conocimiento en el modelo.

**Propuesta de solución:** reforzar la instrucción de reconocer incertidumbre para datos específicos (fechas, lugares, nombres). No se puede eliminar el problema pero sí reducir la frecuencia con que el modelo presenta datos incorrectos como si fueran ciertos.

```
Para datos específicos como fechas, lugares, nombres o cifras: si tenés
alguna duda sobre la precisión del dato, aclaralo antes de responder.
Preferí decir "creo que fue..." antes que afirmar algo con falsa seguridad.
```

---

## Resumen de cambios propuestos al Modelfile

| Cambio | Prioridad | Tipo |
|--------|-----------|------|
| Instrucción explícita para premisas falsas + ejemplo | Alta | Nuevo bloque en SYSTEM |
| Instrucción para pedidos directos de tarea | Media | Nuevo bloque en SYSTEM |
| Refuerzo de incertidumbre en datos específicos | Baja | Modificación de restricción existente |

---

## Próximos pasos

1. Implementar los cambios en `~/Modelfile`
2. Crear `mistral-es-test4`
3. Ejecutar test-04 con preguntas que reproduzcan exactamente los casos fallidos
4. Si pasa → promover a `mistral-es` y cerrar issue #3
