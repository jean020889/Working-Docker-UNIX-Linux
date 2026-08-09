

```markdown
# 🚀 Multi-Language Docker Development Suite

Repositorio unificado con configuraciones profesionales de **Docker**, **Docker Compose** y **VS Code Dev Containers** optimizadas para despliegue y desarrollo continuo en **Go**, **Python**, **Rust** y **TypeScript**.

---

## 📋 Tabla de Contenidos

- [Descripción General](#-descripción-general)
- [Estructura del Repositorio](#-estructura-del-repositorio)
- [Comparativa Técnica de Módulos](#-comparativa-técnica-de-módulos)
- [Guía de Inicio Rápido](#-guía-de-inicio-rápido)
  - [Levantar un Servicio Específico](#1-levantar-un-servicio-específico)
  - [Levantar Todo el Stack](#2-levantar-todo-el-stack)
- [Estrategias por Lenguaje](#-estrategias-de-desarrollo-y-producción)
  - [🐹 Go](#-go)
  - [🐍 Python](#-python)
  - [🦀 Rust](#-rust)
  - [📘 TypeScript](#-typescript)
- [VS Code Dev Containers](#-vs-code-dev-containers)
- [Licencia](#-licencia)

---

## 📋 Descripción General

Este repositorio reúne plantillas de arquitectura para desarrollo aislado y producción de alta eficiencia. Garantiza:
- **Hot-Reload en Desarrollo**: Recarga automática sin rebuilds manuales de imágenes.
- **Imágenes de Producción Ultra-Ligeras**: Construcción multi-etapa (*Multi-Stage Build*) utilizando bases como `scratch`, `alpine` o `slim`.
- **Aislamiento Total**: Cero dependencias requeridas en la máquina host (solo Docker).

---


## 📊 Comparativa Técnica de Módulos

| Lenguaje | Motor Hot-Reload (Dev) | Imagen de Producción | Puerto |
| --- | --- | --- | --- |
| **Go** | `cosmtrek/air` | `scratch` / `alpine` | `8080` |
| **Python** | `uvicorn --reload` | `python:3.12-slim` | `8000` |
| **Rust** | `cargo-watch` | `debian:bookworm-slim` | `8081` |
| **TypeScript** | `tsx watch` | `node:20-alpine` | `3000` |

---

## 🚀 Guía de Inicio Rápido

### 1. Levantamiento de Servicios Individuales

Para trabajar únicamente en una tecnología específica:

```bash
# Para Go
docker compose up go-app

# Para Python
docker compose up python-app

# Para Rust
docker compose up rust-app

# Para TypeScript
docker compose up typescript-app

```

### 2. Levantamiento Global

Si deseas iniciar todos los servicios simultáneamente:

```bash
docker compose up -d

```

---

## 🛠️ Estrategias de Desarrollo y Producción

### 🐹 Go

* **Desarrollo**: Monitoreo directo con `air` re-compilando binarios en memoria al guardar cambios.
* **Producción**: Generación de binario estático compilado con `CGO_ENABLED=0` alojado en una imagen `scratch` limpia (tamaño < 15 MB).

### 🐍 Python

* **Desarrollo**: Ejecución sobre `uvicorn` con la bandera `--reload` y montaje de volúmenes en caliente.
* **Producción**: Instalación sin caché (`pip install --no-cache-dir`) sobre una base `python:3.12-slim`.

### 🦀 Rust

* **Desarrollo**: Compilación incremental administrada por `cargo-watch` reutilizando la carpeta `target` mediante volúmenes de Docker.
* **Producción**: Compilación optimizada en modo `--release` dentro de la etapa builder y binario extraído a una imagen de runtime reducida.

### 📘 TypeScript

* **Desarrollo**: Ejecución directa de TypeScript sin paso de compilación visible mediante `tsx watch`.
* **Producción**: Compilación multi-etapa con `tsc`, aislando las `devDependencies` y ejecutando el JavaScript generado en `dist/`.

---

## 💻 VS Code Dev Containers

Cada directorio incluye soporte completo para **Dev Containers**. Para iniciar tu entorno en un contenedor aislado:

1. Instala la extensión **Dev Containers** en VS Code.
2. Abre la paleta de comandos (`Ctrl + Shift + P` o `Cmd + Shift + P`).
3. Selecciona **Dev Containers: Reopen in Container**.
4. Elige el servicio deseado para trabajar con las herramientas específicas del lenguaje ya instaladas.

---

## 📄 Licencia

Licencia **MIT**. Libre para uso, modificación y distribución comercial o personal.

```

```
