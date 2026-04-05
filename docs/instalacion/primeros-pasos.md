# Guía: Configuración de IA Local en Ubuntu

## Introducción

Esta guía explica cómo instalar y configurar un entorno completo de inteligencia artificial en local sobre Ubuntu, sin depender de servicios en la nube. Todo el procesamiento ocurre en tu propia PC, lo que garantiza privacidad y acceso sin internet.

El stack que vamos a instalar consiste en dos componentes principales:

- **Ollama:** motor que descarga y ejecuta los modelos de lenguaje
- **Open WebUI:** interfaz gráfica web para chatear con los modelos

---

## Hardware utilizado

| Componente | Detalle |
|-----------|---------|
| CPU | AMD Ryzen 2400G (APU con Radeon Vega 11 integrada) |
| RAM | 16 GB (compartida entre CPU y GPU integrada) |
| Sistema Operativo | Ubuntu |

> Al tratarse de una APU, la GPU comparte la RAM del sistema. Esto permite
> correr modelos de hasta 7B parámetros sin GPU dedicada.

---

## Parte 1: Ollama

### ¿Qué es Ollama?

Ollama es una herramienta open source (licencia MIT) que permite descargar y ejecutar modelos de lenguaje (LLMs) en tu propia PC. Funciona como un servidor local al que se pueden conectar distintas interfaces, incluyendo la terminal y Open WebUI.

Repositorio oficial: https://github.com/ollama/ollama

### Instalación

Abrí la terminal (`Ctrl + Alt + T`) y ejecutá:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Descarga del modelo Mistral

Mistral es un modelo open source con licencia Apache 2.0, desarrollado por Mistral AI. Es liviano, eficiente y no tiene restricciones comerciales. Es ideal para hardware con CPU o GPU AMD.

```bash
ollama pull mistral
```

> Ocupa aproximadamente 4.4 GB en disco.

### Personalización: modelo en español

Por defecto Mistral responde en varios idiomas. Para que responda siempre en español se crea una versión personalizada usando un Modelfile.

**¿Qué es un Modelfile?** Es un archivo de configuración que define el comportamiento del modelo, similar a un Dockerfile. Permite establecer instrucciones permanentes sin tener que repetirlas en cada conversación.

**Crear el Modelfile:**

```bash
cat > ~/Modelfile << 'EOF'
FROM mistral
SYSTEM "Eres un asistente útil y amigable. Responde siempre en español, sin importar el idioma en que te hablen."
EOF
```

**Crear el modelo personalizado:**

```bash
ollama create mistral-es -f ~/Modelfile
```

**Verificar que se creó correctamente:**

```bash
ollama list
```

Deberías ver dos entradas:

```
NAME                 SIZE
mistral-es:latest    4.4 GB
mistral:latest       4.4 GB
```

> Aunque aparecen como 4.4 GB cada uno, Ollama comparte los archivos base
> entre ambos. En disco no se duplica el espacio.

**Usar el modelo:**

```bash
ollama run mistral-es
```

Para salir de la sesión de chat escribí `/bye`.

### Comandos útiles de Ollama

| Comando | Descripción |
|--------|-------------|
| `ollama list` | Ver modelos instalados |
| `ollama pull <modelo>` | Descargar un modelo |
| `ollama run <modelo>` | Iniciar chat en la terminal |
| `ollama ps` | Ver modelos activos en memoria |
| `ollama stop <modelo>` | Detener un modelo |

### Modelos recomendados para este hardware

| Modelo | RAM usada | Licencia | Ideal para |
|--------|----------|----------|------------|
| `mistral` (7B) | ~8 GB | Apache 2.0 | Chat general, redacción |
| `llama3.2` (3B) | ~4 GB | Llama Community | Chat rápido |
| `phi3` (3.8B) | ~4 GB | MIT | Código, razonamiento |

---

## Parte 2: Open WebUI

### ¿Qué es Open WebUI?

Open WebUI es una interfaz gráfica web de código abierto que se conecta a Ollama y permite chatear con los modelos desde el navegador, con una experiencia similar a ChatGPT. Se ejecuta como un contenedor Docker y funciona completamente en local.

Repositorio oficial: https://github.com/open-webui/open-webui

### Requisitos previos

Tener Docker instalado y corriendo:

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

Agregar tu usuario al grupo Docker para no necesitar `sudo`:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Instalación

```bash
docker run -d -p 3010:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

**Descripción de los parámetros:**

| Parámetro | Descripción |
|-----------|-------------|
| `-d` | Ejecuta el contenedor en segundo plano |
| `-p 3010:8080` | Expone la interfaz en el puerto 3010 |
| `--add-host` | Permite que Docker acceda a servicios del sistema anfitrión |
| `-v open-webui:/app/backend/data` | Persiste los datos entre reinicios |
| `--restart always` | Inicia automáticamente al encender la PC |

> Se usa el puerto 3010 porque los puertos 3000 al 3005 estaban ocupados.

Acceder desde el navegador:

```
http://localhost:3010
```

La primera vez pedirá crear un usuario local (nombre, email y contraseña). Este usuario es solo para la interfaz y no se conecta a internet.

### Problema: "No hay modelos disponibles"

**Causa:** Docker no puede acceder a `localhost` del sistema anfitrión directamente. Por defecto Ollama escucha solo en `localhost`, lo que lo hace inaccesible desde dentro del contenedor.

**Solución:** configurar Ollama para que escuche en todas las interfaces de red:

```bash
sudo systemctl edit ollama
```

Agregar en el editor:

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```

Guardar con `Ctrl+O` → Enter → `Ctrl+X`. Luego reiniciar Ollama:

```bash
sudo systemctl restart ollama
```

Verificar que Ollama sigue respondiendo:

```bash
curl http://localhost:11434
# Respuesta esperada: Ollama is running
```

Refrescar `http://localhost:3010`. Los modelos ya aparecen en el selector de la parte superior de la pantalla.

---

## Resumen del stack

```
Navegador
    │
    ▼
Open WebUI (http://localhost:3010)
    │
    ▼
Ollama (http://localhost:11434)
    │
    ▼
Modelo: mistral-es
```

---

## Solución de problemas frecuentes

| Problema | Causa | Solución |
|---------|-------|----------|
| Open WebUI no muestra modelos | Ollama no es accesible desde Docker | Configurar `OLLAMA_HOST=0.0.0.0` |
| El modelo responde en otro idioma | Sin instrucción de idioma | Crear modelo con Modelfile |
| Ollama no responde | Servicio detenido | `sudo systemctl restart ollama` |
| Open WebUI no carga | Contenedor detenido | `docker start open-webui` |
