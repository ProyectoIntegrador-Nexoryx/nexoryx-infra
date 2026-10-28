# NEXORYX — Infraestructura

Plataforma web de estudio y colaboración en tiempo real (Proyecto Integrador, Univalle).

Este repositorio **no contiene código de la aplicación**. Su función es orquestar los tres servicios, cada uno en su propio repositorio, para levantarlos juntos con Docker Compose.

| Servicio | Repositorio | Tecnología | Producción |
| --- | --- | --- | --- |
| Frontend | [Nexoryx_frontend](https://github.com/ProyectoIntegrador-Nexoryx/Nexoryx_frontend) | React + Vite + Tailwind | https://nexoryx-frontend.vercel.app (Vercel) |
| Backend API | [Nexoryx_backend-api](https://github.com/ProyectoIntegrador-Nexoryx/Nexoryx_backend-api) | NestJS + Firebase Admin | https://nexoryx-backend-api.onrender.com/api/docs (Render, Docker) |
| Backend Realtime | [Nexoryx_backend-realtime](https://github.com/ProyectoIntegrador-Nexoryx/Nexoryx_backend-realtime) | NestJS + Socket.IO | https://nexoryx-backend-realtime.onrender.com/health (Render, Docker) |

```
Navegador ──► Frontend (Vercel) ──HTTP──► Backend API (Render) ──Admin SDK──► Firebase Auth / Firestore
                     └────────WebSocket──► Backend Realtime (Render)
```

El frontend nunca consulta Firestore directamente: todo pasa por los backends, que verifican el ID token de Firebase.

## Requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) abierto y en ejecución.
- Git.
- Acceso al proyecto de Firebase (para obtener las credenciales).

## Levantar todo con Docker Compose

1. Clona este repositorio y, **dentro de él**, los tres servicios:

   ```bash
   git clone https://github.com/ProyectoIntegrador-Nexoryx/nexoryx-infra.git NEXORYX
   cd NEXORYX
   git clone https://github.com/ProyectoIntegrador-Nexoryx/Nexoryx_frontend.git frontend
   git clone https://github.com/ProyectoIntegrador-Nexoryx/Nexoryx_backend-api.git backend-api
   git clone https://github.com/ProyectoIntegrador-Nexoryx/Nexoryx_backend-realtime.git backend-realtime
   ```

   Los nombres de carpeta (`frontend`, `backend-api`, `backend-realtime`) deben ser exactamente esos: `docker-compose.yml` los usa.

2. Crea los archivos de entorno a partir de los ejemplos y completa los valores:

   ```bash
   cp .env.example .env
   cp backend-api/.env.example backend-api/.env
   cp backend-realtime/.env.example backend-realtime/.env
   ```

   | Archivo | Qué contiene |
   | --- | --- |
   | `.env` | Configuración web pública de Firebase (`VITE_FIREBASE_*`), que se incrusta en el frontend al construir la imagen. |
   | `backend-api/.env` | Credenciales de la cuenta de servicio de Firebase (`FIREBASE_*`) y `FRONTEND_URL`. |
   | `backend-realtime/.env` | Credenciales de Firebase y `FRONTEND_URL`. |

   En ambos backends, `FRONTEND_URL` debe incluir `http://localhost:8080` (el frontend en Docker) para que CORS lo permita.

3. Construye y levanta los contenedores:

   ```bash
   docker compose up --build
   ```

   Compose espera a que ambos backends respondan su health check antes de arrancar el frontend.

4. Abre:

   | Servicio | URL |
   | --- | --- |
   | Frontend | http://localhost:8080 |
   | API (Swagger) | http://localhost:3000/api/docs |
   | API health | http://localhost:3000/api/v1/health |
   | Realtime health | http://localhost:3001/health |

Para detener todo: `Ctrl + C` y luego `docker compose down`.

## Despliegue

- **Backends (Render):** cada repositorio tiene su `Dockerfile`; Render construye la imagen y ejecuta el contenedor en cada push a `main`. Las variables de entorno se configuran en el panel de Render.
- **Frontend (Vercel):** Vercel compila Vite en cada push a `main`. Las variables `VITE_*` se configuran en el panel de Vercel.

Ninguna credencial se guarda en los repositorios: solo los archivos `.env.example` sin valores reales.

## Equipo y flujo de trabajo

- Una rama por historia (`feature/US-XX-descripcion` o `feature/TS-XX-descripcion`).
- Pull request hacia `main`; el administrador revisa y hace merge.
