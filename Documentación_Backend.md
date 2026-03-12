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
| 22/02/26 | c41f5b8 | 2nd migration: remove epp inventory | Se eliminaron los modelos ProductoEPP e InventarioReal de `tables.py` y `base.py`. Se generó la migración Alembic 1249f8c750da que elimina las tablas productos_epp e inventario de la base de datos. |
| 23/02/26 | 109240d | user table and migration | Se agregó el modelo User en `tables.py` con campos de autenticación (email, hashed_password) y datos básicos (nombre, area, activo, fecha_creacion). Se registró en `base.py` y se generó la migración Alembic 25988f7c9a68 que crea la tabla users con índice único en email. |
| 23/02/26 | 04e308b | jwt login config | Se implementó el sistema de autenticación JWT. Se agregó `security.py` para hashing con bcrypt y generación/validación de tokens. Se creó el endpoint `/auth/login` para autenticación y `/auth/me` protegido con get_current_user. También se añadieron los schemas de autenticación y dependencias (python-jose, passlib[bcrypt], email-validator). |
| 23/02/26 | d020c86 | fix bcrypt version | Se fijaron las versiones de passlib[bcrypt]==1.7.4 y bcrypt==4.0.1 para resolver incompatibilidad entre ambas librerías. |
| 24/02/26 | 38e8368 | utils: generate emails | Se agregó el archivo `utils.py` con funciones utilitarias para normalizar textos y generar correos corporativos automáticamente a partir del nombre y apellidos del colaborador, usando el dominio de la empresa. |
| 25/02/26 | 770485f | areas & colaboradores tables | Se reestructuró el modelo de datos: se expandió Colaborador con campos laborales completos, se agregaron tablas Area, ColaboradorAreaHistorial y PeriodoMesFiscal, y se actualizó User con rol, colaborador_id y debe_cambiar_password. |
| 25/02/26 | 1951adf | areas & colaboradores trables migration | Se generó la migración Alembic `b91515838bb8` correspondiente al commit anterior: crea las tablas areas, periodos_mes_fiscal, usuarios_odoo y colaborador_area_historial, reestructura colaboradores y actualiza users con los nuevos campos. |
| 25/02/26 | 61eca2f | fix: user.area->user.rol | Se corrigió el payload del JWT, reemplazando el campo area por rol al generar el token en `/auth/login`. |
| 25/02/26 | d375a6f | fix: auth user fields | Se actualizó UserResponse reemplazando area y nombre por rol y debe_cambiar_password, y agregando colaborador_id como campo opcional. |
| 25/02/26 | 860b6c3 | colaboradores first endpoints | Se implementó el CRUD completo de colaboradores: schemas Pydantic, operaciones en `crud/colaborador.py` (crear, editar, baja y cambio de área con historial) y endpoints REST protegidos por rol. Además, se extrajeron get_current_user y get_current_admin a `dependencies.py`. |
| 04/03/26 | 38489d3 | auth loads user + colaborador info | Se actualizó `/auth/me` para cargar el colaborador y su área con joinedload y devolver los datos del perfil (nombres, apellidos, cargo, area, celular) directamente en UserResponse. |
| 04/03/26 | 8137f10 | fix: auth import | Se corrigió el import de User, apuntando a app.models.tables en lugar de app.models. |
| 05/03/26 | 76667c3 | user auth + front cors | Se agregó endpoint `/auth/crear-usuario` (solo admin) que genera email corporativo y crea el usuario con contraseña temporal. Se agregaron los schemas CrearUsuarioRequest y CrearUsuarioResponse. Se ampliaron los orígenes CORS para incluir Vite dev server y variantes de la IP de Tailscale. |
| 05/03/26 | ebaab20 | fix: auth usuario as default | Se cambió el rol por defecto al crear usuarios de **"vendedor"** a **"usuario".** |
| 06/03/26 | 9fc31c0 | add fecha_ingreso & tipo_vinculo | Se agregaron **fecha_ingreso** y **tipo_vinculo** al schema UserResponse y a la respuesta del endpoint `/auth/me.` |
| 06/03/26 | 66c0525 | auth> add email_personal | Se agregó **email_personal** al schema UserResponse y a la respuesta del endpoint `/auth/me`. |
| 06/03/26 | 717593a | fix: email_personal -> colaborador.email | Se corrigió el **campo email_personal** para leer **colaborador.email** en lugar de **colaborador.email_personal** |
| 06/03/26 | 3c5f9d8 | security docs gitignore | Se agregó **SECURITY_AUDIT.md** al `.gitignore` para evitar subir documentación de auditoría de seguridad al repositorio. |
| 06/03/26 | f190c21 | kpi_ventas tables | Se agregaron 6 modelos para el registro detallado de KPIs de ventas: **RegistroLlamada**, **RegistroVisita**, **RegistroClienteNuevo**, **RegistroTiempoCotizacion**, **RegistroGasto** y **RegistroFacturacion**, todos vinculados a Colaborador. |
| 06/03/26 | 6b4270c | kpi_ventas schemas | Se creó el archivo `kpi.py` con los schemas Pydantic para los KPIs de ventas, incluyendo modelos para llamadas, visitas, clientes nuevos, tiempo de cotización, gastos, facturación y un resumen de KPIs. |
| 06/03/26 | 2fc032e | crud: kpi_ventas | Se creó `crud/kpi.py` con operaciones de crear, eliminar y listar por período para los 6 KPIs, más get_resumen_periodo que agrega conteos, sumas y promedio de tiempo de cotización en una sola consulta. |
| 06/03/26 | 2c01c21 | kpi_ventas endpoints | Se creó `endpoints/kpis.py` con rutas GET, POST y DELETE para los KPIs y /resumen para el agregado del período. Los endpoints obtienen colaborador_id desde el usuario autenticado y se registró el router con el prefijo `/kpis`. |
| 06/03/26 | 7865b92 | migration: kpi_ventas tables | Se generó la migración Alembic `f685fe587b3e` que crea las 6 tablas de registro de KPIs de ventas. |
| 06/03/26 | c0da093 | crud: area | Se creó `schemas/area.py` con los schemas **AreaCreate**, **AreaUpdate** y **AreaResponse**, este último incluyendo total_colaboradores como campo calculado. |
| 06/03/26 | fc84a00 | crud & schema: area | Se creó `crud/area.py` con operaciones para listar áreas (con conteo de colaboradores activos), crear y editar áreas. |
| 06/03/26 | 57425ba | areas endpoints | Se creó `endpoints/areas.py` con rutas GET, POST y PATCH para gestión de áreas, y se registró el router bajo el prefijo /areas. |
| 06/03/26 | ------- | endpoint: users | - |
| 06/03/26 | ------- | add tiene_usuario->colaborador | - |
| 06/03/26 | ------- | endpoint: historial_area_colaborador | - |
| 06/03/26 | ------- | kpi tables & migration | - |
| 06/03/26 | ------- | endpoint + crud + schema: kpi_meta | - |
| 06/03/26 | ------- | periodo_fiscal->semana_fiscal & migration | - |
| 03/03/26 | ------- | --- | --- |
| 03/03/26 | ------- | --- | --- |
| 03/03/26 | ------- | --- | --- |
| 03/03/26 | ------- | --- | --- |
| 03/03/26 | ------- | --- | --- |
| 03/03/26 | ------- | --- | --- |
| 03/03/26 | ------- | --- | --- |
| 03/03/26 | ------- | --- | --- |
| 03/03/26 | ------- | --- | --- |

## Resumen Final
- **Período:** 03/03/2026 – 10/03/2026 (- días)  
- **Total de Commits:** 15 (incluye setup inicial y mejoras)  
- **Observación:** Los commits reflejan cambios reales y necesarios en el proyecto, se trabajó con claridad, corrigiendo o ajustando partes del proyecto.


