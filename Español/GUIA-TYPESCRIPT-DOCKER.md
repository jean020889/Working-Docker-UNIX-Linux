# GUÍA UNIVERSAL: ENTORNO TYPESCRIPT PROFESIONAL CON DOCKER Y VS CODE

A diferencia de los lenguajes compilados a binario nativo (como Go o Rust), TypeScript requiere un paso de transpilación a JavaScript (`tsc`) antes de ser ejecutado por Node.js. En desarrollo, el enfoque profesional evita transpilar manualmente en cada cambio utilizando motores de ejecución directa en memoria con *Hot-Reload* como `tsx`.

---

## Requisito Previo Universal: Instalación por Distribución

Lo único del flujo general que varía entre distribuciones Linux no está en la configuración de Docker o TypeScript, sino en **cómo instalas el motor de Docker, Docker Compose y VS Code en tu sistema** antes de empezar:

| Acción / Paso | Linux Mint / Ubuntu / Debian | Fedora / RHEL | Arch Linux / Manjaro |
| --- | --- | --- | --- |
| **Instalar Docker** | `sudo apt install docker.io` | `sudo dnf install docker-ce` | `sudo pacman -S docker` |
| **Instalar Docker Compose** | `sudo apt install docker-compose-plugin` | `sudo dnf install docker-compose-plugin` | `sudo pacman -S docker-compose` |
| **Habilitar el servicio** | `sudo systemctl enable --now docker` | `sudo systemctl enable --now docker` | `sudo systemctl enable --now docker` |
| **Permisos de usuario** | `sudo usermod -aG docker $USER` | `sudo usermod -aG docker $USER` | `sudo usermod -aG docker $USER` |

> **Nota:** Tras agregar tu usuario al grupo `docker`, debes reiniciar la sesión o ejecutar `newgrp docker` para aplicar los permisos sin necesidad de anteponer `sudo`.

---

## 1. Creación del Proyecto Base en TypeScript

Ejecuta este bloque en tu terminal para estructurar la aplicación base con Fastify y TypeScript:

```bash
# 1. Crear carpeta del proyecto e ingresar
mkdir -p ~/mi-proyecto-typescript && cd ~/mi-proyecto-typescript

# 2. Crear package.json
cat << 'EOF' > package.json
{
  "name": "mi-app-typescript",
  "version": "1.0.0",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "tsx watch src/index.ts"
  },
  "dependencies": {
    "fastify": "^4.28.1"
  },
  "devDependencies": {
    "@types/node": "^20.14.9",
    "typescript": "^5.5.2",
    "tsx": "^4.16.0"
  }
}
EOF

# 3. Crear tsconfig.json
cat << 'EOF' > tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"]
}
EOF

# 4. Crear el código fuente (src/index.ts)
mkdir -p src
cat << 'EOF' > src/index.ts
import Fastify from 'fastify';

const fastify = Fastify({ logger: true });

fastify.get('/', async (request, reply) => {
  return { status: 'ok', message: '¡Servidor TypeScript corriendo en Docker con Hot-Reload!' };
});

const start = async () => {
  try {
    await fastify.listen({ port: 8080, host: '0.0.0.0' });
    console.log('Servidor iniciado en http://localhost:8080');
  } catch (err) {
    fastify.log.error(err);
    process.exit(1);
  }
};

start();
EOF

```

---

## 2. Construcción del Dockerfile Multi-Stage Optimizado

Crea un archivo llamado `Dockerfile` en la raíz del proyecto. Este archivo separa la instalación de dependencias, la transpilación de TypeScript a JavaScript y la ejecución final limpia:

```dockerfile
# --- ETAPA 1: Construcción (Build)
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json tsconfig.json ./
RUN npm ci
COPY src ./src
RUN npm run build

# --- ETAPA 2: Producción (Limpia)
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist

EXPOSE 8080
CMD ["node", "dist/index.js"]

```

---

## 3. Construcción y Prueba Única (`docker run`)

```bash
# Construir la imagen
docker build -t mi-app-typescript .

# Ejecutar el contenedor
docker run -d -p 8080:8080 --name servidor-typescript mi-app-typescript

# Probar en la terminal
curl http://localhost:8080

```

---

## 4. Finalizar o Reiniciar Contenedores en Conflicto

```bash
# Ver qué contenedor usa el puerto 8080
docker ps --filter "publish=8080"

# Detener y remover el contenedor específico
docker stop servidor-typescript
docker rm servidor-typescript

# O liberar todos los servicios si usas docker-compose
docker compose down

```

---

## 5. Flujo de Trabajo en VS Code con Hot-Reload (`tsx`)

Para desarrollar sin tener que reconstruir la imagen de Docker ni transpilar manualmente en cada cambio de código, utilizamos `tsx` dentro del contenedor con montaje de volúmenes.

### Paso 1: Crear el `Dockerfile.dev`

Crea el archivo `Dockerfile.dev` para el entorno de desarrollo:

```dockerfile
FROM node:20-alpine
WORKDIR /app

COPY package*.json tsconfig.json ./
RUN npm install

CMD ["npm", "run", "dev"]

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
      - /app/node_modules
    environment:
      - NODE_ENV=development

```

Abre el proyecto en VS Code (`code .`) y levántalo ejecutando en la terminal integrada:

```bash
docker compose up

```

Cualquier cambio guardado en `src/index.ts` se reflejará al instante gracias al motor de monitoreo de `tsx` sin reiniciar el contenedor.

---

## 6. Dev Containers (Entorno Aislado Completo)

Instala las extensiones **Dev Containers** y **ESLint** en VS Code.

Crea el archivo `.devcontainer/devcontainer.json`:

```json
{
  "name": "Entorno TypeScript Docker",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:1-20-bullseye",
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode"
      ]
    }
  },
  "forwardPorts": [8080]
}

```

Presiona `Ctrl + Shift + P` y selecciona **Dev Containers: Reopen in Container**.
