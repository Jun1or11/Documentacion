# Documentación de Commits / Intranet_backend_

## Introducción
-----------------------------------------------------------------------------------------------------------------

## Configuración inicial del frontend

-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------
| Fecha | Commit | Título | Descripción |
|------|------|------|------|
| 18/02/26 | c0da4ec | initial commit: estructura base + backend | Se creó la estructura base del proyecto backend. Se configuró FastAPI con CORS, endpoint de salud y documentación bajo el prefijo /api. Se definieron los modelos SQLAlchemy (Colaborador, ProductoEPP, InventarioReal) con creación automática de tablas al iniciar. Se implementaron dos endpoints para colaboradores: sincronización desde Odoo vía XML-RPC y creación manual. Se configuró la conexión a PostgreSQL con pool de conexiones y manejo de sesiones. Se containerizó todo con Docker Compose incluyendo Nginx como proxy inverso, PostgreSQL con secretos en archivos, y el backend FastAPI, usando una red interna aegis_network sin exponer el backend directamente. |
| 18/02/26 | e3b11ef | demo test | Se agregó un archivo demo.md de prueba. |
| 19/02/26 | 6dd7e68 | python version upgrade -> 3.12 | Se actualizó la imagen base de Docker del backend de `python:3.11-slim` a `python:3.12-slim.` |
| 22/02/26 | f5d8083 | alembic config | Se inicializó Alembic como sistema de migraciones. Se agregaron `alembic.ini`, `env.py` y el template `script.py.mako` con la configuración estándar para una base de datos única. Se montaron los archivos de Alembic en el contenedor del backend vía `docker-compose.yml`. Se eliminó `demo.md`. |
| 22/02/26 | 64b77ed | alembic env config + 1st migration | Se configuró `env.py` para conectarlo al proyecto real: se inyecta la DATABASE_URL desde los secrets de Docker en tiempo de ejecución, y se apunta target_metadata a Base.metadata para habilitar el soporte de --autogenerate. Se generó la primera migración initial_schema, que queda vacía dado que las tablas ya existían en la BD. |
| 22/02/26 | c41f5b8 | 2nd migration: remove epp inventory
 | Se eliminaron los modelos ProductoEPP e InventarioReal de `tables.py` y `base.py`. Se generó la migración Alembic 1249f8c750da que elimina las tablas productos_epp e inventario de la base de datos. |
| 23/02/26 | 109240d | user table and migration | Se agregó el modelo User en `tables.py` con campos de autenticación (email, hashed_password) y datos básicos (nombre, area, activo, fecha_creacion). Se registró en `base.py` y se generó la migración Alembic 25988f7c9a68 que crea la tabla users con índice único en email. |

| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
| 03/03/26 | 47d8cc9 | - | - |
## Resumen Final
- **Período:** 03/03/2026 – 10/03/2026 (- días)  
- **Total de Commits:** 15 (incluye setup inicial y mejoras)  
- **Observación:** Los commits reflejan cambios reales y necesarios en el proyecto, se trabajó con claridad, corrigiendo o ajustando partes del proyecto.


