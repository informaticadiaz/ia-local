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
- [Uso diario de Open WebUI](docs/uso/open-webui.md)
- [Uso diario de Ollama](docs/uso/ollama.md)

## Comportamiento del modelo

- [01 — Cómo funciona un LLM](docs/comportamiento/01-como-funciona-un-llm.md)
- [02 — El SYSTEM prompt](docs/comportamiento/02-system-prompt.md)
- [03 — Parámetros del Modelfile](docs/comportamiento/03-parametros-modelfile.md)
- [04 — Prompt engineering](docs/comportamiento/04-prompt-engineering.md)
- [05 — Testing y validación](docs/comportamiento/05-testing-y-validacion.md)
- [06 — Implementación de Amauta](docs/comportamiento/06-amauta-implementacion.md)
- [Propuesta de mejoras al Modelfile](docs/comportamiento/propuesta-mejoras.md)

### Tests
- [Test 01](docs/comportamiento/test/test-01.md)
- [Test 02](docs/comportamiento/test/test-02.md)
- [Test 03](docs/comportamiento/test/test-03.md)
- [Test 04](docs/comportamiento/test/test-04.md)
- [Test 05](docs/comportamiento/test/test-05.md)
- [Test 06](docs/comportamiento/test/test-06.md)

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
