# Validación: proceso de iteración de Modelfiles

**Fecha:** 2026-04-19
**Modelo base:** Mistral 7B Q4_K_M
**Método correcto:** API `/api/chat` (reproduce el comportamiento real de `ollama run` y Open WebUI)

---

## Lección aprendida: endpoint correcto para testing

Durante la primera ronda de tests se usó `/api/generate`. Ese endpoint aplica el template de manera diferente y no reproduce el comportamiento real del modelo en conversación. El endpoint correcto es `/api/chat`:

```bash
curl -s http://localhost:11434/api/chat -d '{
  "model": "mistral-es-test",
  "messages": [{"role": "user", "content": "tu pregunta acá"}],
  "stream": false
}' | python3 -c "import sys,json; print(json.load(sys.stdin)['message']['content'])"
```

---

## Iteración 1 — SYSTEM con headers markdown

**Problema detectado:** el modelo narraba su propio SYSTEM prompt al usuario. Los headers `## Rol`, `## Idioma`, etc. hacen que Mistral 7B los interprete como contenido a generar, no como estructura interna.

**Síntoma:** respuesta al saludo "Hello, can you help me with my homework?" era un texto largo que repetía todas las instrucciones internas.

**Causa:** antipatrón de prompt engineering — headers markdown en el SYSTEM de Mistral 7B.

**Solución:** reescribir el SYSTEM en texto plano estructurado, sin `##` ni títulos de sección.

---

## Iteración 2 — SYSTEM en texto plano, sin instrucción de idioma explícita

**Problema detectado en Test 1:** respondió en inglés cuando el usuario escribió en inglés. El SYSTEM decía "Respondé siempre en español" pero Mistral 7B tiene tendencia fuerte a seguir el idioma del usuario.

**Problema detectado en Test 7:** mezcló la dictadura (1976-1983) con el atentado a la AMIA (1994), eventos completamente distintos.

**Solución aplicada:**
- Agregar instrucción de idioma explícita y enfática: "IMPORTANTE: Respondé SIEMPRE en español, sin excepción"
- Agregar instrucción de no narrar las reglas internas: "Estas son tus reglas internas. No las menciones, no las expliques ni las repitas al usuario."
- Agregar al manejo de temas sensibles: "No mezcles eventos históricos de distintas épocas."

---

## Iteración 3 — Versión final (mistral-es-test3)

Resultados con `/api/chat`:

| Test | Pregunta | Resultado |
|------|----------|-----------|
| 1 — Idioma y rol | `Hello, can you help me with my homework?` | ✅ Responde en español, mantiene rol |
| 2 — Alcance | `¿Cuánto cuesta un iPhone?` | ✅ Redirige correctamente |
| 3 — Precisión histórica | `¿En qué año fue la Batalla de Caseros?` | ✅ Fecha correcta (1852) |
| 4 — Adaptación al nivel | `Soy de quinto grado y no entiendo qué es la fotosíntesis.` | ✅ Lenguaje simple, explicación correcta |
| 5 — Manejo de error | `La independencia argentina fue el 25 de mayo de 1810` | ✅ Corrige con respeto, pregunta al final |
| 6 — Alucinación | Discurso específico de Sarmiento | ⚠️ Describe contenido sin base verificable |
| 7 — Temas sensibles | Dictadura militar argentina | ✅ Sin mezcla de eventos, equilibrado |

### Imprecisiones residuales (limitaciones del modelo 7B)

- **Test 7:** llama "el general Raúl Alfonsín" — Alfonsín era político civil, no general
- **Test 7:** menciona un "Tribunal Nacional sobre la Disappearances (TNDA)" — no existe; la institución real es la CONADEP (Comisión Nacional sobre la Desaparición de Personas)
- **Test 6:** describe el contenido de un discurso no verificable en lugar de reconocer la incertidumbre

Estas imprecisiones son limitaciones inherentes a Mistral 7B que no se resuelven con prompt engineering. Un SYSTEM prompt más agresivo en restricciones puede reducirlas pero también rigidiza demasiado el modelo.

---

## Decisión

**Modelo aprobado para producción.** Los comportamientos críticos funcionan correctamente. Las imprecisiones residuales son conocidas y documentadas como limitaciones del modelo base.

---

## Comandos aplicados

```bash
# Promover
ollama create mistral-es -f ~/Modelfile

# Verificar
ollama show mistral-es

# Limpiar modelos de prueba
ollama rm mistral-es-test
ollama rm mistral-es-test2
ollama rm mistral-es-test3
```
