# Documentación de Commits / Intranet_Frontend_

## Introducción
-----------------------------------------------------------------------------------------------------------------

## Configuración inicial del frontend

-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------

-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------
| Fecha | Commit | Cambios | Título | Descripción |
|------|------|------|------|
| 18/02/26 | c0da4ec | 14 files +421 / -0 | initial commit: estructura base + backend | Se creó la estructura base del proyecto backend. Se configuró FastAPI con CORS, endpoint de salud y documentación bajo el prefijo /api. Se definieron los modelos SQLAlchemy (Colaborador, ProductoEPP, InventarioReal) con creación automática de tablas al iniciar. Se implementaron dos endpoints para colaboradores: sincronización desde Odoo vía XML-RPC y creación manual. Se configuró la conexión a PostgreSQL con pool de conexiones y manejo de sesiones. Se containerizó todo con Docker Compose incluyendo Nginx como proxy inverso, PostgreSQL con secretos en archivos, y el backend FastAPI, usando una red interna aegis_network sin exponer el backend directamente. |
| 18/02/26 | e3b11ef | 1 file +1 / -0 | demo test | Se agregó un archivo demo.md de prueba. |
| 19/02/26 | 6dd7e68 | 1 file +1 / -1 | python version upgrade -> 3.12 | Se actualizó la imagen base de Docker del backend de `python:3.11-slim` a `python:3.12-slim.` |
| 03/03/26 | f5d8083 | 6 files +223 / -1 | alembic config | Se inicializó Alembic como sistema de migraciones. Se agregaron alembic.ini, env.py y el template script.py.mako con la configuración estándar para una base de datos única. Se montaron los archivos de Alembic en el contenedor del backend vía docker-compose.yml. Se eliminó demo.md. |
| 03/03/26 | 47d8cc9 | - | - | - |
| 03/03/26 | 47d8cc9 | - | - | - |
| 03/03/26 | 47d8cc9 | - | - | - |
| 03/03/26 | 47d8cc9 | - | - | - |
| 03/03/26 | 47d8cc9 | - | - | - |
| 03/03/26 | 47d8cc9 | - | - | - |
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
