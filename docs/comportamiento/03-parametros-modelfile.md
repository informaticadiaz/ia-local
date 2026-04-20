# Parámetros del Modelfile

## Qué son los parámetros de inferencia

El SYSTEM prompt define qué hace el modelo. Los parámetros de inferencia definen cómo lo hace: qué tan creativo es, qué tan largo responde, si tiende a repetirse, cuánto contexto puede manejar.

En el Modelfile se definen con la directiva `PARAMETER`:

```
FROM mistral
SYSTEM "..."

PARAMETER temperature 0.3
PARAMETER num_ctx 4096
PARAMETER repeat_penalty 1.1
```

Cada `PARAMETER` afecta el proceso de generación de tokens. Sin parámetros explícitos, el modelo usa los valores por defecto del modelo base — que no siempre son los más adecuados para un caso de uso específico.

---

## Parámetros principales

### `temperature`

Controla qué tan predecible o creativo es el modelo.

- **Valor bajo (0.1–0.3):** respuestas más deterministas, consistentes y directas. El modelo elige casi siempre el token más probable.
- **Valor medio (0.5–0.7):** balance entre coherencia y variedad. Valor por defecto de Mistral.
- **Valor alto (0.8–1.0):** respuestas más variadas, creativas, pero también más propensas a errores o desvíos.

```
PARAMETER temperature 0.3
```

Para un asistente educativo que debe dar respuestas precisas y consistentes, temperatura baja es lo correcto.

---

### `top_p`

Define un umbral de probabilidad acumulada. El modelo solo considera tokens cuya probabilidad acumulada no supere ese valor.

Con `top_p 0.9`, el modelo descarta los tokens menos probables y elige entre los que juntos suman el 90% de probabilidad. Reduce salidas erráticas sin hacer el modelo tan rígido como una temperatura muy baja.

```
PARAMETER top_p 0.9
```

Valor razonable para la mayoría de casos. No es necesario ajustarlo salvo que haya problemas específicos de calidad.

---

### `top_k`

Limita la cantidad de tokens candidatos que el modelo considera en cada paso. Con `top_k 40`, solo evalúa los 40 tokens más probables.

Valores bajos hacen el modelo más conservador. Valores altos dan más diversidad pero también más riesgo de salidas raras.

```
PARAMETER top_k 40
```

Valor por defecto de Mistral. Para uso educativo no hace falta cambiarlo.

---

### `repeat_penalty`

Penaliza la repetición de tokens que ya aparecieron en el contexto reciente. Evita que el modelo entre en bucles o repita frases.

- `1.0` = sin penalización (comportamiento por defecto)
- `1.1` = penalización leve, recomendado para respuestas largas
- `1.3` o más = penalización fuerte, puede hacer las respuestas menos naturales

```
PARAMETER repeat_penalty 1.1
```

---

### `num_ctx`

Define el tamaño máximo del contexto en tokens. Incluye el SYSTEM prompt, el historial de la conversación y el mensaje actual.

El modelo `mistral-es` tiene un contexto máximo de 32.768 tokens. Sin embargo, un contexto grande consume más RAM.

Con 16 GB de RAM compartida entre CPU y GPU:

| `num_ctx` | RAM aproximada (modelo activo) | Recomendado para |
|-----------|-------------------------------|------------------|
| 2048 | ~5 GB | Respuestas cortas, bajo consumo |
| 4096 | ~6 GB | Uso general, conversaciones medianas |
| 8192 | ~8 GB | Conversaciones largas, documentos |

```
PARAMETER num_ctx 4096
```

Para Amauta, conversaciones educativas no suelen ser muy largas. 4096 es suficiente y deja margen de RAM para el sistema.

---

### `num_predict`

Define el máximo de tokens que el modelo puede generar en una sola respuesta. No fuerza respuestas cortas: el modelo puede terminar antes si considera que ya respondió. Solo evita respuestas extremadamente largas.

```
PARAMETER num_predict 1024
```

Para respuestas educativas 1024 tokens (~750 palabras) es más que suficiente.

---

## Parámetros que generalmente no hace falta tocar

| Parámetro | Por qué no tocarlo |
|-----------|--------------------|
| `top_k` | El valor por defecto de Mistral (40) es adecuado |
| `top_p` | 0.9 funciona bien para la mayoría de casos |
| `seed` | Solo sirve para reproducibilidad exacta en debugging |
| `tfs_z` | Tail free sampling — parámetro experimental, no documentado en Ollama |

---

## Perfiles por caso de uso

### Educativo (Amauta)
Respuestas precisas, consistentes y moderadamente detalladas.

```
PARAMETER temperature 0.3
PARAMETER top_p 0.9
PARAMETER repeat_penalty 1.1
PARAMETER num_ctx 4096
PARAMETER num_predict 1024
```

### Creativo (brainstorming, redacción)
Más variedad y originalidad, menos predictibilidad.

```
PARAMETER temperature 0.8
PARAMETER top_p 0.95
PARAMETER repeat_penalty 1.05
PARAMETER num_ctx 4096
PARAMETER num_predict 2048
```

### Técnico (código, análisis)
Alta precisión, respuestas estructuradas.

```
PARAMETER temperature 0.1
PARAMETER top_p 0.85
PARAMETER repeat_penalty 1.1
PARAMETER num_ctx 8192
PARAMETER num_predict 2048
```

---

## Modelfile de ejemplo con parámetros

```
FROM mistral

SYSTEM "Sos un asistente educativo argentino..."

PARAMETER temperature 0.3
PARAMETER top_p 0.9
PARAMETER repeat_penalty 1.1
PARAMETER num_ctx 4096
PARAMETER num_predict 1024
```

Este es el punto de partida para construir el Modelfile completo de Amauta.

---

## Próximo paso

Con el SYSTEM prompt y los parámetros cubiertos, el siguiente documento aborda cómo escribir instrucciones efectivas: técnicas de prompt engineering que mejoran la adherencia del modelo al comportamiento deseado.

→ [04 — Prompt engineering](04-prompt-engineering.md)
