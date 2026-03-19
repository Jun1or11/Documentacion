#  Indusegur Aegis — Backend API


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
---

## 4. Arquitectura del backend

El proyecto sigue una arquitectura en capas bien definida:

```
HTTP Request
    │
    ▼
[Endpoint] → valida schema (Pydantic) → verifica jwt (dependencies.py)
    │
    ▼
[CRUD] → consulta / muta la base de datos vía SQLAlchemy ORM
    │
    ▼
[PostgreSQL] ← schema gestionado por Alembic
```

- **Versionado**: todas las rutas viven bajo `/api/v1/`. El prefijo `/api` lo inyecta Nginx como `root_path` de FastAPI.
- **Autenticación**: Bearer Token (JWT HS256). Dos guards: `get_current_user` (cualquier usuario activo) y `get_current_admin` (solo rol `admin`).
- **Configuración**: `pydantic-settings` lee las variables desde Docker Secrets montados en `/run/secrets/`. Sin archivos `.env` en producción.
- **Migraciones**: Alembic gestiona el esquema. Las tablas no se crean con `create_all` en runtime.

---

## 5. Flujo de una petición autenticada

```
Cliente (React)
    │
    │  HTTP  Authorization: Bearer <jwt>
    ▼
Nginx (proxy)  →  añade prefijo /api  →  redirige al contenedor backend
    │
    ▼
FastAPI (uvicorn:8000)
    │
    ├─ CORSMiddleware: valida origen
    ├─ Router v1: determina handler
    ├─ dependencies.py: decodifica JWT → obtiene User de BD
    ├─ Pydantic schema: valida payload del request
    │
    ▼
Función CRUD  →  SQLAlchemy  →  PostgreSQL
    │
    ▼
Response JSON serializado por Pydantic
```

---

## 6. Instalación y ejecución
### Con Docker
```bash
# Clonar el repositorio
git clone <url-del-repositorio>

# 1. Ir al proyecto
cd indusegur-intranet-main

# 2. Configurar secrets 
mkdir C:\opt\aegis-secrets

# Crear archivos necesarios
echo postgres> C:\opt\aegis-secrets\db_user.txt
echo 123456> C:\opt\aegis-secrets\db_password.txt
echo http://localhost> C:\opt\aegis-secrets\odoo_url.txt
echo test> C:\opt\aegis-secrets\odoo_db.txt
echo admin> C:\opt\aegis-secrets\odoo_user.txt
echo admin> C:\opt\aegis-secrets\odoo_password.txt

# 3. Construir y levantar servicios
docker compose up -d --build

# 4. Verificar contenedores
docker ps

# 5. Ver logs (opcional)
docker compose logs -f

```
Probar en el navegador:
API base: http://localhost:8080

Documentación (Swagger): http://localhost:8080/api/docs

### En local (sin Docker)
```bash
# Clonar el repositorio
git clone <url-del-repositorio>

# 1. Ir al proyecto
cd indusegur-intranet-main

# 2. Entrar al backend
cd backend

# 3. Crear entorno virtual
python -m venv venv

# 4. Activar entorno virtual
venv\Scripts\activate

# 5. Instalar dependencias
pip install -r requirements.txt

# 6. Configurar variables de entorno
$env:DB_USER="postgres"
$env:DB_PASSWORD="123456"
$env:DB_HOST="localhost"
$env:DB_PORT="5432"
$env:DB_NAME="test_db"

$env:ODOO_URL="http://localhost"
$env:ODOO_DB="test"
$env:ODOO_USER="admin"
$env:ODOO_PASSWORD="admin"

# 7. Ejecutar servidor
uvicorn app.main:app --reload
```

Probar en el navegador:
API base: http://127.0.0.1:8000

Documentación (Swagger): http://127.0.0.1:8000/docs

