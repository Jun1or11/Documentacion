# Documentación de Commits / Intranet_backend_

## Introducción
En esta sección se documentan los principales commits realizados durante el desarrollo del backend del proyecto. Cada commit describe los cambios implementados en el servidor, incluyendo la configuración inicial, definición de modelos, migraciones de base de datos, endpoints REST y correcciones a lo largo del proceso.

## Configuración inicial del backend

El primer commit corresponde a la configuración inicial del backend. Se estableció la estructura base utilizando FastAPI con soporte CORS, documentación automática y un endpoint de salud bajo el prefijo /api. Se definieron los primeros modelos con SQLAlchemy y se configuró la conexión a PostgreSQL con manejo de sesiones y pool de conexiones.
También se containerizó el proyecto desde el inicio usando Docker Compose, integrando Nginx como proxy inverso y PostgreSQL con secretos en archivos, todo bajo una red interna aegis_network sin exponer el backend directamente al exterior.

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
| 06/03/26 | 2b767f7 | endpoint: users | Se agregaron los endpoints **GET /auth/usuarios** y **PATCH /auth/usuarios/{user_id}** (solo admin) para listar y actualizar rol o estado de usuarios. Se añadieron los schemas **UsuarioListResponse** y **UsuarioUpdateRequest**. |
| 06/03/26 | d819bf3 | add tiene_usuario->colaborador | Se agregó el campo tiene_usuario a ColaboradorResponse y se añadió el helper _set_tiene_usuario en `crud/colaborador.py` que lo calcula consultando la tabla users |
| 06/03/26 | b9ba57d | endpoint: historial_area_colaborador | Se agregó el endpoint **GET /{colaborador_id}/historial-areas** (solo admin) que retorna el historial de cambios de área de un colaborador con nombre, color y fechas. |
| 06/03/26 | 9913996 | kpi tables & migration | Se generó la migración Alembic `0c6959b25899` que crea las tablas kpi_definiciones y kpi_metas. Se agregaron los modelos KPIDefinicion y KPIMeta a `tables.py`. |
| 06/03/26 | 771ce69 | endpoint + crud + schema: kpi_meta | Se crearon `schemas/kpi_meta.py`, `crud/kpi_meta.py` y `endpoints/kpi_metas.py` para gestionar definiciones y metas de KPI. Se añadió upsert_meta para crear o actualizar metas por período y se registró el router kpi_metas con el prefijo /kpi |
| 06/03/26 | d9452a9 | periodo_fiscal->semana_fiscal & migration | Se generaron cuatro migraciones de Alembic (`7063b1c213a0`, `a312eb5e4a11`, `48fdbb437a80` y `333afcb2a45f`) para reemplazar la tabla de períodos fiscales. Como parte del cambio, se sustituyó el modelo PeriodoMesFiscal por SemanaFiscal en `tables.py` y `base.py`, pasando de períodos mensuales a semanas fiscales por trimestre. |
| 07/03/26 | 63d295c | fix: semanas_fiscales endpoints | Se crearon `schemas/semana_fiscal.py`, `crud/semana_fiscal.py` y `endpoints/semanas_fiscales.py` con endpoints para consultar la semana actual, listar trimestres y semanas, generar un trimestre a partir de tres meses y eliminarlo. Además, se registró el router bajo el prefijo /semanas-fiscales. |
| 07/03/26 | 4cf18cf | fix: kpi_metas format & endpoints & migration | Se generó la migración Alembic `fe8b13b82f47` para renombrar el campo año a anio en el modelo KPIMeta. También se actualizó este cambio en schemas y crud. Además, se agregó el endpoint **GET /metas/area/{area_id}** que devuelve los colaboradores activos del área con sus metas. |
| 09/03/26 | 687117e | fix: colaborador_id missing + dashboard endpoint | Se agregó colaborador_id faltante en la respuesta de `/auth/me`. Se crearon los endpoints **GET /kpis/dashboard** y **GET /kpis/dashboard/equipo** que devuelven KPIs de la semana actual, acumulado trimestral y metas para un colaborador o el equipo completo de un área |
| 11/03/26 | 93a18a6 | fix: kpi gastos sumar->promediar | Se corrigió el cálculo de gastos en los endpoints de dashboard, cambiando de **sum** a **avg** tanto para la vista personal como la de equipo. |
| 11/03/26 | ab079c0 | fix: numeric_types:int->dec | Se generó la migración Alembic `bfb4670cc780` que cambia campos de **INTEGER** a **Numeric(12,2)** y agrega tipo_agregacion en **kpi_definiciones**. También se actualizaron modelos y schemas de **int** a **Decimal**. |
| 12/03/26 | a135f2a | fix: metas_equipo tables & enpoints | --- |
| 12/03/26 | 453b719 | add nombre_colaborador to gastos | --- |


## Resumen Final
- **Período:** 18/02/2026 – 12/03/2026 (- días)  
- **Total de Commits:** 44 (incluye setup inicial y mejoras)  
- **Observación:** *El backend presenta una base funcional con autenticación, gestión de colaboradores, áreas y KPIs implementados. El desarrollo fue iterativo con varias correcciones a lo largo del período*.


