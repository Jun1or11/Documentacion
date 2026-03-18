#  Indusegur Aegis — Backend API

> Sistema central de la Intranet Indusegur · REST API · v1.0.0

---

## 1. Descripción general

Backend de la **Intranet Indusegur (Aegis)**, construido con **FastAPI** y **PostgreSQL**. Expone una API REST versionada (`/api/v1`) que gestiona colaboradores, áreas organizativas, KPIs de ventas y autenticación con JWT.

El sistema está desplegado con **Docker Compose** y servido detrás de un proxy **Nginx**, con secretos gestionados mediante Docker Secrets (sin `.env` en producción).

---

## 2. Stack tecnológico

| Capa | Tecnología | Versión |
|---|---|---|
| Framework API | FastAPI | 0.110.3 |
| Servidor ASGI | Uvicorn | 0.32.0 |
| ORM | SQLAlchemy | 2.0.36 |
| Base de datos | PostgreSQL | 16 (Alpine) |
| Driver DB | psycopg2-binary | 2.9.10 |
| Migraciones | Alembic | 1.13.0 |
| Autenticación | python-jose (JWT) + passlib (bcrypt) | — |
| Configuración | pydantic-settings | 2.8.1 |
| Formatos de entrada | python-multipart | 0.0.20 |
| Lenguaje | Python | 3.12 (slim) |
| Proxy reverso | Nginx | Alpine |
| Contenedores | Docker + Docker Compose | — |

---

## 3. Estructura del proyecto

```
indusegur-intranet-main/
│
├──  docker-compose.yml        # Orquestación de todos los servicios
├──  nginx/                    # Configuración del proxy reverso
│
├──  frontend/                 # Aplicación React (servida por Nginx)
│
└──   backend/
    ├── Dockerfile               # Imagen Python 3.12-slim
    ├── requirements.txt         # Dependencias del proyecto
    ├── alembic.ini              # Configuración de migraciones
    ├── alembic/                 # Migraciones de base de datos
    │
    └── app/                     # Código fuente principal
        ├── main.py              # Punto de entrada, CORS, routers
        ├── dependencies.py      # Guards de autenticación (JWT)
        │
        ├── core/                # Configuración y seguridad transversal
        ├── db/                  # Sesión de base de datos (SQLAlchemy)
        ├── models/              # Tablas ORM (SQLAlchemy models)
        ├── schemas/             # Contratos de entrada/salida (Pydantic)
        ├── crud/                # Consultas y lógica de base de datos
        ├── services/            # Servicios externos (ej. Odoo)
        └── api/
            └── v1/
                ├── api.py       # Registro central de todos los routers
                └── endpoints/   # Un archivo por módulo funcional
```
