# ia-local

Documentación y configuración para correr modelos de inteligencia artificial de forma local, sin depender de servicios en la nube.

## Hardware objetivo

- **CPU:** AMD Ryzen 2400G (APU con Radeon Vega 11 integrada)
- **RAM:** 16 GB
- **OS:** Ubuntu

## Stack utilizado

| Herramienta | Descripción | Licencia |
|-------------|-------------|----------|
| [Ollama](https://ollama.com) | Motor para correr modelos LLM en local | MIT |
| [Open WebUI](https://github.com/open-webui/open-webui) | Interfaz web tipo ChatGPT | MIT |
| [Mistral 7B](https://mistral.ai) | Modelo de lenguaje principal | Apache 2.0 |

## Documentación

- [Primeros pasos — instalación y configuración](docs/instalacion/primeros-pasos.md)

## Inicio rápido

```bash
# Instalar Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Descargar modelo
ollama pull mistral

# Levantar interfaz web
docker run -d -p 3010:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

Accedé a la interfaz en `http://localhost:3010`.
