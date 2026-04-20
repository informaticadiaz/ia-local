# Uso diario de Ollama

## Estado del servicio

Ollama corre como servicio del sistema y se inicia automáticamente al encender la PC.

Verificar que está activo:

```bash
systemctl status ollama
```

Reiniciar si hay problemas:

```bash
sudo systemctl restart ollama
```

Confirmar que responde:

```bash
curl http://localhost:11434
# Respuesta esperada: Ollama is running
```

---

## Comandos frecuentes

| Comando | Descripción |
|--------|-------------|
| `ollama list` | Ver modelos instalados |
| `ollama ps` | Ver modelos cargados en memoria |
| `ollama run <modelo>` | Iniciar chat en la terminal |
| `ollama pull <modelo>` | Descargar o actualizar un modelo |
| `ollama show <modelo>` | Ver configuración y system prompt del modelo |
| `ollama stop <modelo>` | Descargar modelo de la memoria |
| `ollama rm <modelo>` | Eliminar un modelo del disco |

---

## Modelos instalados

```bash
ollama list
```

Modelos actuales:

| Nombre | Tamaño | Descripción |
|--------|--------|-------------|
| `mistral-es:latest` | 4.4 GB | Mistral 7B con instrucción en español y enfoque educativo argentino |
| `mistral:latest` | 4.4 GB | Mistral 7B base |

---

## Chat desde la terminal

```bash
ollama run mistral-es
```

Para salir de la sesión escribir `/bye`.

---

## Modelfiles — crear o actualizar un modelo personalizado

Un Modelfile define el comportamiento del modelo: modelo base, instrucciones permanentes y parámetros.

### Ver el system prompt de un modelo existente

```bash
ollama show mistral-es
```

### Crear o actualizar un Modelfile

Crear el archivo:

```bash
nano ~/Modelfile
```

Estructura básica:

```
FROM mistral
SYSTEM "Instrucción permanente que el modelo seguirá en toda conversación."
```

Aplicar los cambios y crear el modelo:

```bash
ollama create mistral-es -f ~/Modelfile
```

> Si el modelo ya existe, este comando lo reemplaza con la nueva versión.

Verificar que se creó correctamente:

```bash
ollama show mistral-es
```

---

## API local

Ollama expone una API REST en `http://localhost:11434`. Útil para scripts o para verificar el estado desde otras herramientas.

Listar modelos:

```bash
curl http://localhost:11434/api/tags
```

Enviar un mensaje:

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "mistral-es",
  "prompt": "¿Qué fue el Virreinato del Río de la Plata?",
  "stream": false
}'
```

---

## Gestión del disco

Ver cuánto ocupa cada modelo:

```bash
ollama list
```

Eliminar un modelo que no se usa:

```bash
ollama rm mistral
```

> Aunque `ollama list` muestra tamaños individuales, los modelos derivados del mismo base comparten archivos en disco. Eliminar `mistral` no afecta a `mistral-es`.

---

## Configuración del servicio

Ollama está configurado para escuchar en todas las interfaces de red, lo que permite que Open WebUI (Docker) se conecte a él.

Ver la configuración activa:

```bash
cat /etc/systemd/system/ollama.service.d/override.conf
```

Valor actual:

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```
