# Documentación de Commits / Intranet_Frontend_

## Introducción
En esta sección se documentan los principales commits realizados durante el desarrollo del frontend del proyecto. 
Cada commit describe los cambios realizados en el código, incluyendo configuraciones iniciales, creación de componentes, 
implementación de páginas y modificaciones en la estructura del proyecto.

## Configuración inicial del proyecto

El primer commit corresponde a la configuración inicial del proyecto frontend. 
Se estableció la estructura base utilizando **Vite, React y TypeScript**, además de configuraciones necesarias 
para el desarrollo como ESLint y los archivos de configuración del proyecto.

También se crearon los primeros componentes y páginas del sistema, como el **Layout principal**, 
la página de **Dashboard** con KPIs de ventas usando datos simulados (mock data), y la página de **Login**. 
Adicionalmente, se configuró el sistema de **ruteo de la aplicación** y un cliente inicial para la conexión con la API.

----------------------------------------------------------------------------------------------------------
| Fecha | Commit | Título | Descripción |
|------|------|------|------|
| 03/03/26 | adae8df | Setup inicial - layout, dashboard KPI ventas, mock data | Configuración inicial del proyecto frontend, incluyendo estructura base, componentes principales, páginas iniciales, sistema de rutas y datos simulados para los KPIs de ventas. Implementación de layout responsivo y diseño del dashboard para visualización de indicadores de desempeño. |
| 03/03/26 | 47d8cc9 | gitignore upd | Se actualizó el archivo `.gitignore` agregando exclusiones para variables de entorno (.env) y archivos de caché generados por herramientas como ESLint y TypeScript, con el objetivo de evitar que estos archivos temporales o sensibles sean subidos al repositorio. |
| 03/03/26 | 50ffe7d | dashboard fixes + perfil | Se realizaron mejoras en la interfaz del dashboard y se implementó la nueva página de perfil de usuario. Se ajustaron estilos y colores en el componente Layout, se mejoraron gráficos y visualización de KPIs en el Dashboard, y se creó la página `/Perfil.tsx` con información del colaborador usando datos mock. Además, se actualizó el router para permitir la navegación hacia la nueva vista de perfil. |
| 03/03/26 | 49e56e7 | dashboard views + configuracion | Se expandió el dashboard con múltiples vistas de análisis y se implementó la sección de configuración general de la aplicación. Se agregaron componentes para visualizar diferentes perspectivas de datos de ventas y se creó la estructura base para las opciones de configuración. |
| 03/04/26 | f39e4f0 | upd mock data | Actualización y mejora de los datos simulados utilizados en toda la aplicación. Se revisaron y optimizaron las estructuras de datos mock para reflejar de manera más realista los KPIs de ventas y la información de usuarios. |
| 03/06/26 | 5de6728 | login connections + users config | Implementación de conexiones de autenticación y gestión de configuración de usuarios. Se integró lógica de login con endpoints de API, validación de credenciales y se desarrolló la sección de administración de usuarios dentro del módulo de configuración. |
| 03/06/26 | 06ab0f7 | readme docs | Creación de documentación completa en archivo README.md incluyendo instrucciones de instalación, configuración del proyecto, estructura de carpetas, guía de desarrollo y documentación de componentes principales. |
| 03/06/26 | 908e5e2 | perfil: connected to api | Conexión de la página de perfil de usuario con endpoints reales de la API. Se reemplazaron los datos mock por llamadas HTTP a la API, implementando manejo de errores y estado de carga en la interfaz de perfil. |
| 03/06/26 | ab0c601 | registro kpis page | Desarrollo de nueva página dedicada al registro y visualización de KPIs con validación de datos. Se implementó formulario para entrada de KPIs, tabla de histórico de registros y gráficos de tendencias de desempeño. |
| 03/06/26 | 19449f2 | area connection → config | Integración de conexión de áreas administrativas con el módulo de configuración. Se agregó funcionalidad para gestionar diferentes áreas de la organización y su mapeo en la sección de configuración. |
| 03/06/26 | 31b913d | users list in config |  Implementación de lista de usuarios en el módulo de configuración con opciones de gestión (Create, Read, Update). Se agregó una tabla de usuarios que muestra información como nombre, correo, rol, área y estado. También se incorporaron acciones administrativas para crear usuarios, cambiar su rol y activar o desactivar cuentas. |
| 03/07/26 | 7dd2eb3 | admin page: colaboradores management | Desarrollo de página completa de administración para gestión de colaboradores. Se incluye funcionalidad de creación, edición, eliminación de registros de colaboradores y asignación de roles y permisos. |
| 03/07/26 | 2f44d75 | periodos_fiscales tab in config | Adición de pestaña para administración de períodos fiscales dentro del módulo de configuración. Se implementó calendario de períodos, validación de fechas y asociación con KPIs. |
| 03/07/26 | 2a789fd | metas tab in config | Implementación de pestaña de gestión de metas en el módulo de configuración. Se agregó funcionalidad para crear, editar y monitorear metas operacionales con seguimiento de progreso y alertas de cumplimiento. |

----------------------------------------------------------------------------------
