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

## 5. Componentes Principales

### 5.1 Endpoints


####  Auth — `/auth`

| Método | Ruta                  | Acceso | Descripción                            |
|--------|-----------------------|--------|----------------------------------------|
| POST   | `/auth/login`         | Público | Autentica y devuelve JWT              |
| GET    | `/auth/me`            | Usuario | Datos del usuario autenticado         |
| POST   | `/auth/crear-usuario` | Admin   | Crea cuenta para un colaborador       |
| GET    | `/auth/usuarios`      | Admin   | Lista todos los usuarios              |
| PATCH  | `/auth/usuarios/{id}` | Admin   | Modifica rol o estado de un usuario   |

####  Colaboradores — `/colaboradores`

| Método | Ruta                                         | Acceso | Descripción                        |
|--------|----------------------------------------------|--------|------------------------------------|
| GET    | `/colaboradores/`                            | Usuario | Lista colaboradores               |
| GET    | `/colaboradores/{id}`                        | Usuario | Obtiene un colaborador            |
| POST   | `/colaboradores/`                            | Admin   | Crea colaborador                  |
| PATCH  | `/colaboradores/{id}`                        | Admin   | Edita datos del colaborador       |
| PATCH  | `/colaboradores/{id}/baja`                   | Admin   | Da de baja al colaborador         |
| PATCH  | `/colaboradores/{id}/area`                   | Admin   | Cambia área del colaborador       |
| GET    | `/colaboradores/{id}/historial-areas`        | Admin   | Historial de cambios de área      |

####  KPIs — `/kpis`

| Método | Ruta                          | Acceso | Descripción                                     |
|--------|-------------------------------|--------|-------------------------------------------------|
| GET    | `/kpis/resumen`               | Usuario | Resumen KPI por período                        |
| GET    | `/kpis/dashboard`             | Usuario | Dashboard KPI individual (semana + trimestre)  |
| GET    | `/kpis/dashboard/equipo`      | Usuario | Dashboard KPI del equipo con ranking           |
| GET    | `/kpis/llamadas`              | Usuario | Lista llamadas por período                     |
| POST   | `/kpis/llamadas`              | Usuario | Registra llamada                               |
| DELETE | `/kpis/llamadas/{id}`         | Usuario | Elimina llamada                                |
| GET    | `/kpis/visitas`               | Usuario | Lista visitas por período                      |
| POST   | `/kpis/visitas`               | Usuario | Registra visita                                |
| DELETE | `/kpis/visitas/{id}`          | Usuario | Elimina visita                                 |
| GET    | `/kpis/clientes-nuevos`       | Usuario | Lista clientes nuevos por período              |
| POST   | `/kpis/clientes-nuevos`       | Usuario | Registra cliente nuevo                         |
| DELETE | `/kpis/clientes-nuevos/{id}`  | Usuario | Elimina registro                               |
| GET    | `/kpis/tiempo-cotizacion`     | Usuario | Lista tiempos de cotización                    |
| POST   | `/kpis/tiempo-cotizacion`     | Usuario | Registra tiempo de cotización                  |
| DELETE | `/kpis/tiempo-cotizacion/{id}`| Usuario | Elimina registro                               |
| GET    | `/kpis/gastos`                | Usuario | Lista gastos (`?todos=true` para equipo)       |
| POST   | `/kpis/gastos`                | Usuario | Registra gasto                                 |
| DELETE | `/kpis/gastos/{id}`           | Usuario | Elimina gasto                                  |
| GET    | `/kpis/facturacion`           | Usuario | Lista facturación por período                  |
| POST   | `/kpis/facturacion`           | Usuario | Registra facturación                           |
| DELETE | `/kpis/facturacion/{id}`      | Usuario | Elimina registro                               |

####  Áreas — `/areas`

| Método | Ruta           | Acceso | Descripción          |
|--------|----------------|--------|----------------------|
| GET    | `/areas/`      | Usuario | Lista áreas         |
| POST   | `/areas/`      | Admin   | Crea área           |
| PATCH  | `/areas/{id}`  | Admin   | Edita área          |

####  KPI Metas — `/kpi`

| Método | Ruta                                | Acceso | Descripción                          |
|--------|-------------------------------------|--------|--------------------------------------|
| GET    | `/kpi/definiciones`                 | Usuario | Lista definiciones de KPIs          |
| GET    | `/kpi/metas`                        | Admin   | Lista metas por período y área      |
| GET    | `/kpi/metas/colaborador/{id}`       | Usuario | Metas de un colaborador             |
| POST   | `/kpi/metas`                        | Admin   | Crea o actualiza meta (upsert)      |
| DELETE | `/kpi/metas/{id}`                   | Admin   | Elimina meta                        |
| GET    | `/kpi/metas/area/{id}`              | Admin   | Metas de toda el área               |

####  Semanas Fiscales — `/semanas-fiscales`

| Método | Ruta                              | Acceso | Descripción                           |
|--------|-----------------------------------|--------|---------------------------------------|
| GET    | `/semanas-fiscales/actual`        | Usuario | Semana fiscal en curso               |
| GET    | `/semanas-fiscales/{anio}`        | Usuario | Trimestres del año                   |
| GET    | `/semanas-fiscales/{anio}/{trim}` | Usuario | Semanas de un trimestre              |
| POST   | `/semanas-fiscales/generar`       | Admin   | Genera semanas fiscales de trimestre |
| DELETE | `/semanas-fiscales/{anio}/{trim}` | Admin   | Elimina un trimestre completo        |

####  Endpoints de sistema

| Método | Ruta      | Descripción                              |
|--------|-----------|------------------------------------------|
| GET    | `/`       | Health check básico (status + version)  |
| GET    | `/health` | Verifica conectividad con la base de datos |

---


## 6. Flujo de una petición autenticada

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
 
## 7. Base de datos
 
- **Motor:** PostgreSQL 16 (Alpine)
- **Nombre de la BD:** `aegis_intranet`
- **ORM:** SQLAlchemy 2.0 (estilo declarativo)
- **Migraciones:** Alembic
 
### Entidades / Modelos principales (`app/models/tables.py`)
 
**Usuarios & Personas**
 
| Modelo | Descripción |
|--------|-------------|
| `Colaborador` | Información base de la persona (nombres, cargos, vínculos, etc.). |
| `User` | Credenciales corporativas vinculadas lógicamente a un `Colaborador`. |
| `ColaboradorOdoo` | Sincronización de registros directos desde Odoo. |
| `ColaboradorAreaHistorial` | Rastreo de movimientos y vigencia de áreas de un empleado a través del tiempo. |
 
**Catálogos & Tiempo**
 
| Modelo | Descripción |
|--------|-------------|
| `Area` | Agrupación lógica dentro de la empresa. |
| `SemanaFiscal` | Mapeo estricto del tiempo en años y trimestres. |
 
**Indicadores (KPIs)**
 
| Modelo | Descripción |
|--------|-------------|
| `KPIDefinicion` | Estructura de definición del KPI. |
| `KPIMeta` | Valor objetivo de la revisión. |
| Registros eventuales | Tablas satélite como `RegistroLlamada`, `RegistroGasto`, `RegistroFacturacion`, `RegistroVisita`, etc., unidas jerárquicamente al colaborador respectivo. |
 
---

```
## 5. Manejo de errores
 
Los errores están manejados explícitamente y centralizados mediante el sistema de excepciones nativo de FastAPI.
 
- **Tipo de respuesta:** Se lanzan excepciones del tipo `HTTPException` importadas de `fastapi`.
- **Formato devuelto:**
 
```json
{
  "detail": "Mensaje técnico o descriptivo sobre el fallo."
}
```
 
**Códigos HTTP utilizados:**
 
| Código | Significado | Ejemplo de uso |
|--------|-------------|----------------|
| `400 Bad Request` | Incoherencias solicitadas por el cliente | "Área ya existente" |
| `401 Unauthorized` | Sin login o rol inválido | Acceso sin token |
| `403 Forbidden` | Privilegios insuficientes | Ruta solo para admins |
| `404 Not Found` | Registro no encontrado | ID inexistente en DB |

---

## 8. Variables de Entorno
 
Las credenciales se proveen exclusivamente mediante **Docker Secrets** montados en `/run/secrets/`. No se usa archivo `.env` en producción.
 
| Secret File | Variable mapeada | Descripción |
|-------------|------------------|-------------|
| `db_user.txt` | `DB_USER` | Usuario de PostgreSQL |
| `db_password.txt` | `DB_PASSWORD` | Contraseña de PostgreSQL |
| `odoo_url.txt` | `ODOO_URL` | URL del servidor Odoo |
| `odoo_db.txt` | `ODOO_DB` | Base de datos de Odoo |
| `odoo_user.txt` | `ODOO_USER` | Usuario de Odoo |
| `odoo_password.txt` | `ODOO_PASSWORD` | Contraseña de Odoo |
 
> Los archivos de secrets deben existir en el servidor en `/opt/aegis-secrets/` antes de ejecutar Docker Compose.
 
Variables con valores por defecto (no requieren secret):
 
| Variable | Default |
|----------|---------|
| `DB_HOST` | `db` |
| `DB_PORT` | `5432` |
| `DB_NAME` | `aegis_intranet` |
| `TAILSCALE_IP` | `100.76.137.2` |

---

## 9. Manejo de autenticación
 
El backend protege sus rutas mediante **OAuth2 con JWT (JSON Web Tokens)**.
 
- **Generación y firma:** Se utiliza la librería `jose` con una `SECRET_KEY` simétrica bajo el algoritmo **HMAC (HS256)**.
- **Seguridad de contraseñas:** Las contraseñas se almacenan mediante hashing iterativo con **bcrypt** (`passlib`).
 
### Flujo de acceso
 
1. El cliente envía sus credenciales al endpoint `/api/auth/login`.
2. La respuesta incluye un `access_token` temporal (expiración: **8 horas** por defecto).
3. Los accesos subsiguientes transmiten dicho token en el header HTTP:
   ```
   Authorization: Bearer <token>
   ```
 
**Dependencias protegidas:**
- `get_current_user` — valida el payload del token y devuelve al usuario de la DB.
- `get_current_admin` — derivado del anterior; asegura roles administrativos al acceder.
 
---

## 10. Instalación y ejecución
### Con Docker
```bash
# Clonar el repositorio
git clone <url-del-repositorio>

# 1. Ir al proyecto
cd indusegur-intranet-main

# 2. Configurar secrets (EN WSL / UBUNTU)
sudo mkdir -p /opt/aegis-secrets

# Crear archivos necesarios
echo postgres | sudo tee /opt/aegis-secrets/db_user.txt
echo 123456 | sudo tee /opt/aegis-secrets/db_password.txt
echo http://localhost | sudo tee /opt/aegis-secrets/odoo_url.txt
echo test | sudo tee /opt/aegis-secrets/odoo_db.txt
echo admin | sudo tee /opt/aegis-secrets/odoo_user.txt
echo admin | sudo tee /opt/aegis-secrets/odoo_password.txt

# 3. Construir y levantar servicios
docker compose up -d --build

# 4. Verificar contenedores
docker ps

# 5. Ver logs (opcional)
docker compose logs -f

# 6. Verificar contenedores
docker ps

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

