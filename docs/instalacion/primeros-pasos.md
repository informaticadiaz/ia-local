# Configuración de IA Local en Ubuntu

## Hardware utilizado

- **CPU:** AMD Ryzen 2400G (APU con Radeon Vega 11 integrada)
- **RAM:** 16 GB (compartida entre CPU y GPU integrada)
- **OS:** Ubuntu

---

## 1. Instalación de Ollama

Ollama es una herramienta open source (licencia MIT) para correr modelos de lenguaje en local.

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

---

## 2. Descarga del modelo Mistral

Mistral es un modelo open source con licencia Apache 2.0, ideal para hardware con CPU/AMD.

```bash
ollama pull mistral
```

---

## 3. Personalización del modelo (respuesta en español)

### Crear el Modelfile

```bash
cat > ~/Modelfile << 'EOF'
FROM mistral
SYSTEM "Eres un asistente útil y amigable. Responde siempre en español, sin importar el idioma en que te hablen."
EOF
```

### Crear el modelo personalizado

```bash
ollama create mistral-es -f ~/Modelfile
```

### Usar el modelo

```bash
ollama run mistral-es
```

---

## 4. Interfaz gráfica con Open WebUI

Open WebUI provee una interfaz web similar a ChatGPT que se conecta a Ollama.

### Levantar el contenedor Docker

```bash
docker run -d -p 3010:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

### Acceder desde el navegador

```
http://localhost:3010
```

> El flag `--restart always` hace que Open WebUI arranque automáticamente al iniciar la PC.

---

## Comandos útiles de Ollama

| Comando | Descripción |
|--------|-------------|
| `ollama list` | Ver modelos instalados |
| `ollama pull <modelo>` | Descargar un modelo |
| `ollama run <modelo>` | Iniciar chat con un modelo |
| `ollama ps` | Ver modelos corriendo |
| `ollama stop <modelo>` | Detener un modelo |

---

## Modelos recomendados para este hardware

| Modelo | RAM usada | Licencia | Ideal para |
|--------|----------|----------|------------|
| `mistral` (7B) | ~8 GB | Apache 2.0 | Redacción, chat general |
| `llama3.2` (3B) | ~4 GB | Llama Community | Chat rápido |
| `phi3` (3.8B) | ~4 GB | MIT | Código, razonamiento |
