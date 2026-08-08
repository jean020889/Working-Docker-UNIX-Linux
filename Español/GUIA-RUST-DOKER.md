# GUÍA UNIVERSAL: ENTORNO RUST PROFESIONAL CON DOCKER Y VS CODE



Compilar Rust en Docker tiene un reto principal: recompilar dependencias no modificadas en cada `docker build` destruye la velocidad de desarrollo. El enfoque profesional en Rust utiliza `cargo-chef` para cachear dependencias y `cargo-watch` para recompilaciones incrementales ultra rápidas.

---

## Requisito Previo Universal: Instalación por Distribución

Lo único del flujo general que varía entre distribuciones Linux no está en la configuración de Docker o Rust, sino en **cómo instalas el motor de Docker, Docker Compose y VS Code en tu sistema** antes de empezar:

| Acción / Paso | Linux Mint / Ubuntu / Debian | Fedora / RHEL | Arch Linux / Manjaro |
| --- | --- | --- | --- |
| **Instalar Docker** | `sudo apt install docker.io` | `sudo dnf install docker-ce` | `sudo pacman -S docker` |
| **Instalar Docker Compose** | `sudo apt install docker-compose-plugin` | `sudo dnf install docker-compose-plugin` | `sudo pacman -S docker-compose` |
| **Habilitar el servicio** | `sudo systemctl enable --now docker` | `sudo systemctl enable --now docker` | `sudo systemctl enable --now docker` |
| **Permisos de usuario** | `sudo usermod -aG docker $USER` | `sudo usermod -aG docker $USER` | `sudo usermod -aG docker $USER` |

> **Nota:** Tras agregar tu usuario al grupo `docker`, debes reiniciar la sesión o ejecutar `newgrp docker` para aplicar los permisos sin necesidad de anteponer `sudo`.

---

## 1. Creación del Proyecto Base en Rust



Ejecuta este bloque en tu terminal para estructurar la aplicación base utilizando Axum y Tokio:

```bash
# 1. Crear carpeta del proyecto e ingresar
mkdir -p ~/mi-proyecto-rust && cd ~/mi-proyecto-rust

# 2. Crear Cargo.toml
cat << 'EOF' > Cargo.toml
[package]
name = "mi-app-rust"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { version = "1.0", features = ["full"] }
axum = "0.7"
EOF

# 3. Crear el código fuente (src/main.rs)
mkdir -p src
cat << 'EOF' > src/main.rs
use axum::{routing::get, Router};
use std::net::SocketAddr;

#[tokio::main]
async fn main() {
    let app = Router::new().route("/", get(|| async { "¡Servidor Rust corriendo en Docker!" }));
    let addr = SocketAddr::from(([0, 0, 0, 0], 8080));
    println!("Servidor Rust iniciado en http://localhost:8080");
    let listener = tokio::net::TcpListener::bind(addr).await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
EOF

```

---

## 2. Construcción del Dockerfile Multi-Stage Optimizado



Crea un archivo llamado `Dockerfile` en la raíz del proyecto. Este archivo utiliza `cargo-chef` para separar la compilación de dependencias del código de tu aplicación, maximizando el uso del caché de capas de Docker:

```dockerfile
# --- ETAPA 1: Preparación del caché de dependencias
FROM lukemathwalker/cargo-chef:latest-rust-1-alpine AS chef
WORKDIR /app

FROM chef AS planner
COPY . .
RUN cargo chef prepare --recipe-path recipe.json

# --- ETAPA 2: Compilación de binarios y crates
FROM chef AS builder
COPY --from=planner /app/recipe.json recipe.json
# Compila solo las dependencias (se almacena en caché de Docker)
RUN cargo chef cook --release --recipe-path recipe.json
COPY . .
# Compila el binario final
RUN cargo build --release --bin mi-app-rust

# --- ETAPA 3: Imagen Final Mínima
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/target/release/mi-app-rust .
EXPOSE 8080
CMD ["./mi-app-rust"]

```

---

## 3. Construcción y Prueba Única (`docker run`)



```bash
# Construir la imagen
docker build -t mi-app-rust .

# Ejecutar el contenedor
docker run -d -p 8080:8080 --name servidor-rust mi-app-rust

# Probar en la terminal
curl http://localhost:8080

```

---

## 4. Finalizar o Reiniciar Contenedores en Conflicto



```bash
# Ver qué contenedor usa el puerto 8080
docker ps --filter "publish=8080"

# Detener y remover el contenedor específico
docker stop servidor-rust
docker rm servidor-rust

# O liberar todos los servicios si usas docker-compose
docker compose down

```

---

## 5. Flujo de Trabajo en VS Code con Hot-Reload (`cargo-watch`)



Para desarrollar sin recompilar desde cero en cada cambio, se utiliza `cargo-watch` junto con volúmenes persistentes para el caché de Cargo y la carpeta `target`.

### Paso 1: Crear el `Dockerfile.dev`

Crea el archivo `Dockerfile.dev` para el entorno de desarrollo:

```dockerfile
FROM rust:1.78-slim
WORKDIR /app

# Instalar cargo-watch para detectar cambios de archivos
RUN cargo install cargo-watch

COPY Cargo.toml ./

# Estructura ligera temporal para pre-compilar crates en dev
RUN mkdir src && echo "fn main() {}" > src/main.rs && cargo build && rm -rf src

CMD ["cargo", "watch", "-x", "run"]

```

### Paso 2: Crear el `docker-compose.yml`

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
      - cargo-cache:/usr/local/cargo/registry
      - target-cache:/app/target

volumes:
  cargo-cache:
  target-cache:

```

Ejecuta `docker compose up` en la terminal de VS Code. Con los volúmenes configurados para `target` y `cargo-cache`, las recompilaciones al presionar `Ctrl + S` tardarán solo fracciones de segundo.

---

## 6. Dev Containers (Entorno Aislado Completo)



Instala las extensiones **Dev Containers** y **rust-analyzer** en VS Code.

Crea el archivo `.devcontainer/devcontainer.json`:

```json
{
  "name": "Entorno Rust Docker",
  "image": "mcr.microsoft.com/devcontainers/rust:1-1-bullseye",
  "customizations": {
    "vscode": {
      "extensions": [
        "rust-lang.rust-analyzer",
        "tamasfe.even-better-toml"
      ]
    }
  },
  "forwardPorts": [8080]
}

```

Presiona `Ctrl + Shift + P` y selecciona **Dev Containers: Reopen in Container**.
