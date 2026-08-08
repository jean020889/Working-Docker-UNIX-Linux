[Seguro] He procesado el documento de Python y generado tanto la versión enriquecida en formato Markdown universal como el archivo de habilidad para reutilizar en cualquier momento.

Aquí tienes el contenido limpio, corregido y adaptado universalmente para todas las distribuciones de Linux:

---

# GUÍA UNIVERSAL: ENTORNO PYTHON PROFESIONAL CON DOCKER Y VS CODE



En Python no compilas un binario estático como en Go; ejecutas un entorno interpretado. El enfoque equivalente para mantener un contenedor optimizado e instantáneo requiere ajustar el manejo del buffer de salida (`PYTHONUNBUFFERED=1`) y utilizar recarga en caliente (*Hot-Reload*) mediante Watchfiles o Uvicorn.

---

## Requisito Previo Universal: Instalación por Distribución

Lo único del flujo general que varía entre distribuciones Linux no está en la configuración de Docker o Python, sino en **cómo instalas el motor de Docker, Docker Compose y VS Code en tu sistema** antes de empezar:

| Acción / Paso | Linux Mint / Ubuntu / Debian | Fedora / RHEL | Arch Linux / Manjaro |
| --- | --- | --- | --- |
| **Instalar Docker** | `sudo apt install docker.io` | `sudo dnf install docker-ce` | `sudo pacman -S docker` |
| **Instalar Docker Compose** | `sudo apt install docker-compose-plugin` | `sudo dnf install docker-compose-plugin` | `sudo pacman -S docker-compose` |
| **Habilitar el servicio** | `sudo systemctl enable --now docker` | `sudo systemctl enable --now docker` | `sudo systemctl enable --now docker` |
| **Permisos de usuario** | `sudo usermod -aG docker $USER` | `sudo usermod -aG docker $USER` | `sudo usermod -aG docker $USER` |

> **Nota:** Tras agregar tu usuario al grupo `docker`, debes reiniciar la sesión o ejecutar `newgrp docker` para aplicar los permisos sin necesidad de anteponer `sudo`.

---

## 1. Creación del Proyecto Base en Python



Ejecuta este bloque en tu terminal para estructurar la aplicación base con FastAPI y Uvicorn:

```bash
# 1. Crear carpeta del proyecto e ingresar
mkdir -p ~/mi-proyecto-python && cd ~/mi-proyecto-python

# 2. Crear requirements.txt con FastAPI y Watchfiles para Hot-Reload
cat << 'EOF' > requirements.txt
fastapi==0.111.0
uvicorn[standard]==0.30.1
watchfiles==0.22.0
EOF

# 3. Crear el código fuente (main.py)
cat << 'EOF' > main.py
from fastapi import FastAPI
import uvicorn

app = FastAPI(title="Servidor Python en Docker")

@app.get("/")
def read_root():
    return {"status": "ok", "message": "Servidor Python corriendo en Docker con Hot-Reload!"}

if __name__ == "__main__":
    uvicorn.run("main:app", host="0.0.0.0", port=8080, reload=True)
EOF

```

---

## 2. Construcción del Dockerfile Optimizado



Crea un archivo llamado `Dockerfile` en la raíz del proyecto:

```dockerfile
FROM python:3.11-slim

# Evita que Python escriba archivos .pyc y fuerza la salida directa a terminal
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

# Copiar e instalar dependencias primero para aprovechar el caché de capas de Docker
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar el código de la aplicación
COPY . .

EXPOSE 8080
CMD ["python", "main.py"]

```

---

## 3. Construcción y Prueba Única (`docker run`)



```bash
# Construir la imagen
docker build -t mi-app-python .

# Ejecutar el contenedor
docker run -d -p 8080:8080 --name servidor-python mi-app-python

# Probar en la terminal
curl http://localhost:8080

```

---

## 4. Finalizar o Reiniciar Contenedores en Conflicto



```bash
# Ver qué contenedor usa el puerto 8080
docker ps --filter "publish=8080"

# Detener y remover el contenedor específico
docker stop servidor-python
docker rm servidor-python

# O liberar todos los servicios si usas docker-compose
docker compose down

```

---

## 5. Flujo de Trabajo en Visual Studio Code



### Opción A: Hot-Reload con Docker Compose (Recomendado)



Crea el archivo `docker-compose.yml` en la raíz de tu proyecto:

```yaml
services:
  web:
    build: .
    ports:
      - "8080:8080"
    volumes:
      - .:/app
    environment:
      - PYTHONUNBUFFERED=1
    command: uvicorn main:app --host 0.0.0.0 --port 8080 --reload

```

Abre el proyecto en VS Code (`code .`) y levántalo ejecutando en la terminal integrada:

```bash
docker compose up

```

Cada vez que modifiques `main.py` y presiones `Ctrl + S`, Uvicorn/Watchfiles recargará el código en milisegundos sin reiniciar el contenedor.

---

### Opción B: Dev Containers (Entorno Aislado Completo)



Instala las extensiones **Dev Containers** y **Python** en VS Code.

Crea el archivo `.devcontainer/devcontainer.json`:

```json
{
  "name": "Entorno Python Docker",
  "image": "mcr.microsoft.com/devcontainers/python:1-3.11-bullseye",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance"
      ]
    }
  },
  "forwardPorts": [8080]
}

```

Presiona `Ctrl + Shift + P` y elige **Dev Containers: Reopen in Container**.
