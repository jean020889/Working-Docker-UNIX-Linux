# GUÍA UNIVERSAL: USO DE DOCKER PARA GO EN LINUX



Docker no se opera como una máquina virtual tradicional donde abres una interfaz y ves un sistema operativo arrancando. Funciona mediante contenedores: procesos aislados que empaquetan tu aplicación con todas sus dependencias.

Para trabajar con Docker solo necesitas entender 4 conceptos básicos y 6 comandos fundamentales.

---

## Los 4 Conceptos Clave

1. **Imagen:** Una plantilla de solo lectura (como una receta o un instalador).


2. **Contenedor:** La instancia viva y en ejecución de una imagen (la comida preparada o la aplicación corriendo).


3. **Docker Hub:** El registro público oficial donde encuentras imágenes listas (*Node.js, Python, PostgreSQL, Nginx*).


4. **Dockerfile:** El archivo de texto donde describes paso a paso cómo construir tu propia imagen.



---

## Requisito Previo Universal: Instalación por Distribución

Lo único del flujo general que varía entre Linux Mint y otras distribuciones no está dentro del documento, sino en **cómo instalas el motor de Docker, Docker Compose y VS Code en tu sistema** antes de empezar:

| Acción / Paso | Linux Mint / Ubuntu / Debian | Fedora / RHEL | Arch Linux / Manjaro |
| --- | --- | --- | --- |
| **Instalar Docker** | `sudo apt install docker.io` | `sudo dnf install docker-ce` | `sudo pacman -S docker` |
| **Instalar Docker Compose** | `sudo apt install docker-compose-plugin` | `sudo dnf install docker-compose-plugin` | `sudo pacman -S docker-compose` |
| **Habilitar el servicio** | `sudo systemctl enable --now docker` | `sudo systemctl enable --now docker` | `sudo systemctl enable --now docker` |
| **Permisos de usuario** | `sudo usermod -aG docker $USER` | `sudo usermod -aG docker $USER` | `sudo usermod -aG docker $USER` |

> **Nota:** Tras agregar tu usuario al grupo `docker`, debes reiniciar la sesión o ejecutar `newgrp docker` para aplicar los permisos sin necesidad de anteponer `sudo` a cada comando.

---

## Flujo de Trabajo Básico (Comandos Fundamentales)

### 1. Buscar y descargar imágenes

No necesitas instalar Node, Python o bases de datos directamente en tu sistema anfitrión; las descargas como imágenes:

```bash
# Descarga la versión oficial de Nginx desde Docker Hub
docker pull nginx

```

### 2. Listar lo que tienes descargado



```bash
docker images

```

### 3. Crear y ejecutar un contenedor



El comando `docker run` es el más importante.

```bash
# Corre un servidor Nginx en segundo plano (-d) y mapea el puerto 8080 de tu PC al puerto 80 del contenedor (-p)
docker run -d -p 8080:80 --name mi-servidor nginx

```

Si abres `http://localhost:8080` en tu navegador, verás la página de bienvenida de Nginx sin haber instalado el servidor web en tu sistema.

### 4. Ver contenedores en ejecución



```bash
# Muestra solo los contenedores activos
docker ps

# Muestra todos los contenedores (activos y detenidos)
docker ps -a

```

### 5. Entrar a la terminal de un contenedor activo



Si necesitas ejecutar comandos dentro del entorno aislado:

```bash
docker exec -it mi-servidor bash

```

(Escribe `exit` para salir sin apagar el contenedor).

### 6. Detener y eliminar



```bash
# Detener el contenedor por nombre o ID
docker stop mi-servidor

# Borrar el contenedor (libera espacio/nombre)
docker rm mi-servidor

# Borrar la imagen descargada
docker rmi nginx

```

---

## Uso Profesional: Dockerizar un Proyecto Propio



Para desarrollar dentro de un entorno aislado sin ensuciar tu sistema operativo, creas un archivo llamado `Dockerfile` en la raíz de tu proyecto.

Ejemplo de `Dockerfile` para una aplicación en Python:

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "main.py"]

```

Comandos para usarlo:

```bash
# 1. Construir la imagen con el nombre 'mi-app'
docker build -t mi-app .

# 2. Ejecutar tu aplicación
docker run -d -p 5000:5000 mi-app

```

---

## Gestión Simplificada: Docker Compose



Cuando tu proyecto requiere varios servicios a la vez (por ejemplo: tu app + una base de datos PostgreSQL + Redis), no ejecutas múltiples `docker run`. Creas un archivo `docker-compose.yml`:

```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secreto

```

Para encender todo el entorno con un solo comando:

```bash
docker compose up -d

```

Para apagar todo:

```bash
docker compose down

```

---

## Ejemplo de Implementación para un Proyecto en Go



Ejecuta este bloque paso a paso en la terminal para crear un proyecto Go funcional y construir la imagen:

```bash
# 1. Crear la carpeta del proyecto e ingresar a ella
mkdir -p ~/mi-proyecto-go
cd ~/mi-proyecto-go

# 2. Inicializar el módulo de Go
cat << 'EOF' > go.mod
module mi-proyecto-go
go 1.22
EOF

# 3. Crear el código fuente (main.go)
cat << 'EOF' > main.go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "¡Servidor Go corriendo en Docker!")
	})
	fmt.Println("Servidor iniciado en http://localhost:8080")
	http.ListenAndServe(":8080", nil)
}
EOF

# 4. Crear el Dockerfile (Multi-stage build para binario ligero)
cat << 'EOF' > Dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod ./
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o main .

FROM alpine:latest
WORKDIR /root/
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]
EOF

# 5. Construir la imagen de Docker
docker build -t mi-app-go .

```

### Probar la Aplicación



Una vez terminada la construcción, inicia el contenedor:

```bash
docker run -d -p 8080:8080 --name servidor-go mi-app-go

```

Verifica la respuesta enviando una petición desde la terminal:

```bash
curl http://localhost:8080

```

---

## Cómo Finalizar el Proceso de un Contenedor / Resolver Conflictos



### 1. Identificar el contenedor en conflicto



Ejecuta el siguiente comando para ver qué contenedor de Docker está usando ese puerto:

```bash
docker ps --filter "publish=8080"

```

### 2. Detener el contenedor activo



Si el contenedor pertenece a otro proyecto de Docker, detén el contenedor específico usando su ID o nombre:

```bash
docker stop <CONTAINER_ID>
# Ejemplo: docker stop 4ee76bbadf41

```

Si es una instancia huérfana de este mismo proyecto o un entorno de pruebas, fuerza la bajada de todos los servicios para liberar los bindings de red retenidos por `docker-proxy`:

```bash
docker compose down

```

### 3. Volver a levantar el servicio



Una vez liberado el proxy, ejecuta nuevamente:

```bash
docker compose up

```

> **Advertencia de Riesgo:** Matar directamente los procesos `docker-proxy` con `kill -9` dejará la tabla de enrutamiento interna del daemon de Docker y la interfaz de red virtual en un estado inconsistente, requiriendo reiniciar el servicio de Docker por completo (`sudo systemctl restart docker`).
> 
> 

---

## Cómo Trabajar o Programar desde Visual Studio Code



El error más común es modificar el código en VS Code y esperar que el contenedor se actualice solo. Si usas el comando tradicional `docker run`, los cambios no se verán reflejados hasta volver a ejecutar `docker build`.

Para programar cómodamente con VS Code existen dos opciones profesionales:

---

### Opción A: Hot-Reload con Docker Compose (La más rápida)



Con este enfoque mantienes los archivos en tu máquina, abres VS Code normalmente y el contenedor recompila automáticamente cada vez que guardas (`Ctrl + S`).

#### 1. Crear el archivo `docker-compose.yml` en la raíz de tu proyecto

:

```bash
cd ~/mi-proyecto-go
cat << 'EOF' > docker-compose.yml
services:
  app:
    image: golang:1.22-alpine
    working_dir: /app
    volumes:
      - .:/app
    ports:
      - "8080:8080"
    command: go run main.go
EOF

```

#### 2. Abrir el proyecto en VS Code

:

```bash
code .

```

#### 3. Iniciar el entorno de desarrollo

:

Abre la terminal integrada de VS Code (`Ctrl + ~`) y ejecuta:

```bash
docker compose up

```

Cualquier cambio guardado en `main.go` se aplicará inmediatamente sin reconstruir la imagen. Para detenerlo, presiona `Ctrl + C` o ejecuta `docker compose down`.

---

### Opción B: Dev Containers (Programar DIRECTAMENTE dentro del contenedor)



Este es el estándar industrial si no deseas tener el entorno de Go ni herramientas instaladas localmente. VS Code "se conecta" al contenedor y usa el compilador y las extensiones directamente desde allí.

#### 1. Instalar extensiones en VS Code

:

Presiona `Ctrl + Shift + X` en VS Code e instala:

* **Dev Containers** (Oficial de Microsoft)


* **Go** (Oficial de Go Team)



#### 2. Crear la configuración de Dev Container

:

```bash
mkdir -p .devcontainer
cat << 'EOF' > .devcontainer/devcontainer.json
{
  "name": "Entorno Go Docker",
  "image": "mcr.microsoft.com/devcontainers/go:1-1.22-bookworm",
  "customizations": {
    "vscode": {
      "extensions": [
        "golang.go"
      ]
    }
  },
  "forwardPorts": [8080]
}
EOF

```

#### 3. Abrir en el contenedor

:

1. Abre la paleta de comandos de VS Code (`Ctrl + Shift + P`).


2. Selecciona **Dev Containers: Reopen in Container**.


3. VS Code se reiniciará cargando el entorno completo dentro del contenedor aislado con Go y sus herramientas (*LSP, Delve, etc.*).



---

## Integración Air en `docker-compose.yml` para Recompilación Instantánea



El comando predeterminado `go run main.go` dentro de Docker no detecta cambios en tiempo real en sistemas de archivos montados desde el sistema huésped. La herramienta estándar para resolver esto es **Air** (un motor de *live-reload* optimizado para Go).

---

### Paso 1: Crear la Configuración de Air (`.air.toml`)



Crea `.air.toml` en la raíz de tu proyecto Go:

```toml
root = "."
testdata_dir = "testdata"
tmp_dir = "tmp"

[build]
  # Comando para compilar el binario
  cmd = "go build -o ./tmp/main ."
  # Binario resultante que Air ejecutará
  bin = "tmp/main"
  # Extensiones a monitorear
  include_ext = ["go", "tpl", "tmpl", "html"]
  # Carpetas ignoradas para evitar bucles de recompilación
  exclude_dir = ["assets", "tmp", "vendor", "testdata"]
  # Retardo tras detectar un cambio antes de recompilar (ms)
  delay = 1000
  stop_on_error = true
  kill_delay = "0s"

[log]
  time = false

[color]
  main = "magenta"
  watcher = "cyan"
  build = "yellow"
  runner = "green"

```

---

### Paso 2: Crear el `Dockerfile.dev`

Crea un `Dockerfile.dev` específico para desarrollo que instale **Air** dentro de la imagen base de Go:

```dockerfile
FROM golang:1.22-alpine
WORKDIR /app

# Instalar Git y el binario de Air (versión oficial mantenida por air-verse)
RUN apk add --no-cache git
RUN go install github.com/air-verse/air@latest

# Copiar archivos de dependencias
COPY go.mod ./
# COPY go.sum ./ # Descomentar si usas paquetes externos
RUN go mod download

# El código fuente se monta por volumen desde docker-compose
CMD ["air", "-c", ".air.toml"]

```

---

### Paso 3: Configurar el `docker-compose.yml`

Actualiza o crea tu archivo `docker-compose.yml` apuntando a `Dockerfile.dev`:

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "8080:8080"
    volumes:
      - .:/app
    environment:
      - CGO_ENABLED=0
    tty: true

```

---

### Paso 4: Ejecutar y Probar



Abre la terminal en tu proyecto y levanta el contenedor compilando el nuevo entorno:

```bash
docker compose up --build

```

**Verificación:**

Abre `main.go` en VS Code, cambia cualquier texto en el mensaje de respuesta y presiona `Ctrl + S`. Verás de inmediato en la terminal la secuencia de Air:

```text
app-1 | building...
app-1 | running...
app-1 | Servidor iniciado en http://localhost:8080

```

> **Nota importante:** Agrega `/tmp` a tu archivo `.gitignore`, ya que Air creará la carpeta `tmp/` en tu proyecto local para almacenar el binario temporal generado durante cada recompilación.
> 
>
