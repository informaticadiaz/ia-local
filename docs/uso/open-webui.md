# Uso diario de Open WebUI

## Acceso

Open WebUI corre como contenedor Docker con reinicio automático. Una vez que la PC está encendida, la interfaz está disponible sin intervención manual.

Acceder desde el navegador:

```
http://localhost:3010
```

Si la página no carga, verificar que el contenedor esté activo:

```bash
docker ps --filter name=open-webui
```

Si aparece como detenido:

```bash
docker start open-webui
```

---

## Inicio de sesión

La primera vez se solicita usuario y contraseña. Las credenciales son locales — no se conectan a ningún servicio externo.

---

## Iniciar una conversación

1. Hacer clic en **New Chat** (esquina superior izquierda)
2. Seleccionar el modelo en el selector superior — usar `mistral-es:latest`
3. Escribir el mensaje en el campo inferior y presionar Enter o el botón de envío

---

## Modelo disponible

| Modelo | Descripción | Cuándo usarlo |
|--------|-------------|---------------|
| `mistral-es:latest` | Mistral 7B configurado para responder siempre en español | Uso general |
| `mistral:latest` | Mistral 7B base sin instrucción de idioma | Si se necesita respuesta en otro idioma |

El modelo recomendado para uso cotidiano es `mistral-es:latest`.

---

## Historial de conversaciones

Las conversaciones quedan guardadas automáticamente en el panel lateral izquierdo, ordenadas por fecha.

Cada conversación recibe un título generado automáticamente a partir del primer mensaje.

---

## Recomenzar una conversación

Para iniciar una consulta nueva sin contexto previo, siempre usar **New Chat**. Continuar en una conversación existente hace que el modelo tenga en cuenta todo el historial anterior, lo que puede afectar las respuestas.

---

## Verificar que Ollama está respondiendo

Open WebUI se conecta a Ollama para procesar los mensajes. Si los modelos no aparecen en el selector o las respuestas fallan, verificar que Ollama esté activo:

```bash
curl http://localhost:11434
# Respuesta esperada: Ollama is running
```

Si no responde:

```bash
sudo systemctl restart ollama
```
