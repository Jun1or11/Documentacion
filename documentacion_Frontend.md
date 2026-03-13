# Documentación de Commits / Intranet_Frontend_

## Introducción
En esta sección se documentan los principales commits realizados durante el desarrollo del frontend del proyecto. 
Cada commit describe los cambios realizados en el código, incluyendo configuraciones iniciales, creación de componentes, 
implementación de páginas y modificaciones en la estructura del proyecto.

## Configuración inicial del frontend

El primer commit corresponde a la configuración inicial del frontend. 
Se estableció la estructura base utilizando **Vite, React y TypeScript**, además de configuraciones necesarias 
para el desarrollo como ESLint y los archivos de configuración del proyecto.

También se crearon los primeros componentes y páginas del sistema, como el Layout principal, 
la página de Dashboard con KPIs de ventas usando datos simulados (mock data), y la página de Login. 
Adicionalmente, se configuró el sistema de rutas de la aplicación y un cliente inicial para la conexión con la API.

----------------------------------------------------------------------------------------------------------
| Fecha | Commit | Título | Descripción |
|------|------|------|------|
| 03/03/26 | adae8df | Setup inicial - layout, dashboard KPI ventas, mock data | Configuración inicial del frontend, incluyendo estructura base, componentes principales, páginas iniciales, sistema de rutas y datos simulados para los KPIs de ventas. Implementación de layout responsivo y diseño del dashboard para visualización de indicadores de desempeño. |
| 03/03/26 | 47d8cc9 | gitignore upd | Se actualizó el archivo `.gitignore` agregando exclusiones para variables de entorno (.env) y archivos de caché generados por herramientas como ESLint y TypeScript, con el objetivo de evitar que estos archivos temporales o sensibles sean subidos al repositorio. |
| 03/03/26 | 50ffe7d | dashboard fixes + perfil | Se realizaron mejoras en la interfaz del dashboard y se implementó la nueva página de perfil de usuario. Se ajustaron estilos y colores en el componente Layout, se mejoraron gráficos y visualización de KPIs en el Dashboard, y se creó la página `/Perfil.tsx` con información del colaborador usando datos mock. Además, se actualizó el router para permitir la navegación hacia la nueva vista de perfil. |
| 03/03/26 | 49e56e7 | dashboard views + configuracion | Expansión del dashboard y creación de la página de configuración. Se unificaron VistaPersonal y VistaEquipo en un solo componente BloqueKpis, se agregó selector de colaborador para admin/gerencia y soporte de datos por equipo. Se creó la página de configuración con 5 tabs (Periodos, Metas, Usuarios, Áreas, Notificaciones) y se habilitó su ruta. |
| 04/03/26 | f39e4f0 | upd mock data | Corrección de datos mock en Layout, Configuracion y Perfil. Se actualizó el nombre y correo del usuario de prueba para que sean consistentes en todos los archivos. |
| 06/03/26 | 5de6728 | login connections + users config | Implementación del sistema de autenticación real. Se creó el store de usuario (userStore) con fetchUsuario y clearUsuario, se conectó el login con la API con manejo de errores, se eliminaron los mocks de usuario en Layout y Dashboard, y se implementó PrivateRoute para proteger rutas y cargar el usuario automáticamente. |
| 06/03/26 | 06ab0f7 | readme docs | Reemplazo del README por defecto de Vite por la documentación técnica del proyecto. Se incluyó stack tecnológico, estructura de carpetas, endpoints integrados, flujo de autenticación, rutas, roles y permisos, e instrucciones de instalación. |
| 06/03/26 | 908e5e2 | perfil: connected to api | Conexión de la página de perfil con datos reales del store de usuario. Se reemplazaron los datos mock por useUserStore, implementando estado de carga y extendiendo el modelo Usuario con los campos de perfil laboral y contacto. |
| 06/03/26 | ab0c601 | registro kpis page | Creación de la página de registro de KPIs conectada a la API. Se implementaron 6 tabs (Llamadas, Visitas, Clientes nuevos, Tiempo de cotización, Gastos y Facturación) con formularios, tablas y eliminación de registros. Se habilitó la ruta /registro apuntando al nuevo componente. |
| 06/03/26 | 19449f2 | area connection → config | Conexión del tab Áreas con la API real. Se reemplazaron los datos mock por llamadas a /areas/ y se agregó CRUD completo (crear, editar, toggle activo/inactivo) con formulario inline, validación y manejo de errores. |
| 06/03/26 | 31b913d | users list in config |  Implementación de la pestaña de usuarios en configuración. Muestra una tabla con nombre, correo, rol, área y estado de cada cuenta, con acciones para crear usuarios vinculados a un colaborador, cambiar su rol y activar/desactivar el acceso. |
| 06/03/26 | 7dd2eb3 | admin page: colaboradores management | Desarrollo de la página de administración de colaboradores con creación, edición, eliminación, asignación de roles y permisos, historial y estadísticas. Se agregó la ruta `/admin` e importación de la página Admin en `index.tsx.` |
| 07/03/26 | 2f44d75 | periodos_fiscales tab in config | Adición de pestaña para administración de períodos fiscales en el módulo de configuración. Se implementó generación y edición de trimestres, cálculo automático de semanas por mes, validación de fechas y visualización de la semana actual. |
| 07/03/26 | 2a789fd | metas tab in config | Conexión del tab Metas con la API real. Se reemplazaron los datos mock por llamadas a /kpi/metas/area/1, se agregó selector de año y trimestre, edición de metas por colaborador con guardado a la API vía POST /kpi/metas y manejo de estado de carga. |
| 10/03/26 | 2a789fd | load kpi data -> dashboard | Conexión del dashboard con la API real. Se reemplazaron los datos mock por llamadas a /kpis/dashboard y /kpis/dashboard/equipo, se agregó carga dinámica de semanas fiscales y colaboradores, skeleton loader durante la carga, y se mejoró el selector de período con navegación por año, trimestre y semana real. |
| 12/03/26 | 2a789fd | fix: metas_equipo views |----|
| 12/03/26 | 2a789fd | fix: configuracion permissions |----|
| 12/03/26 | 2a789fd | hide not working functions & imprvs |----|
| 12/03/26 | 2a789fd | fix: registros dates & gastos equipo view |----|
| 12/03/26 | 2a789fd | registros: date fixes |----|
| 12/03/26 | 2a789fd | unusedlocals->false |----|
## Resumen Final
- **Período:** 03/03/2026 – 10/03/2026 (- días)  
- **Total de Commits:** 15 (incluye setup inicial y mejoras)  
- **Observación:** Los commits reflejan cambios reales y necesarios en el proyecto, se trabajó con claridad, corrigiendo o ajustando partes del proyecto.

