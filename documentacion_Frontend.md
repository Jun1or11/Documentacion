# Documentación de Commits - Proyecto Frontend

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

| Fecha | Commit | Título | Descripción |
|------|------|------|------|
| 03/03/26 | adae8df | Setup inicial - layout, dashboard KPI ventas, mock data | Configuración inicial del proyecto frontend, incluyendo estructura base, componentes principales, páginas iniciales, sistema de rutas y datos simulados para los KPIs de ventas. |
| 03/03/26 | 47d8cc9 | gitignore upd | Se actualizó el archivo `.gitignore` agregando exclusiones para variables de entorno (.env) y archivos de caché generados por herramientas como ESLint y TypeScript, con el objetivo de evitar que estos archivos temporales o sensibles sean subidos al repositorio. |
| 03/03/26 | 50ffe7d | dashboard fixes + perfil | Se realizaron mejoras en la interfaz del dashboard y se implementó la nueva página de perfil de usuario. Se ajustaron estilos y colores en el componente Layout, se mejoraron gráficos y visualización de KPIs en el Dashboard, y se creó la página `/Perfil.tsx` con información del colaborador usando datos mock. Además, se actualizó el router para permitir la navegación hacia la nueva vista de perfil. |
| 09/03/26 | 49e56e7 | dashboard views + configuracion | Se implementó la nueva página **Configuración** para la administración del sistema, incluyendo gestión de periodos fiscales, metas de KPIs, usuarios, áreas y notificaciones utilizando datos mock. También se realizaron mejoras en el Dashboard para permitir diferentes vistas de KPIs (personal, colaborador seleccionado y equipo), agregando lógica de selección de asesor y visualización consolidada del equipo. Además, se actualizaron los datos mock de KPIs y se modificó el router para integrar la nueva ruta `/configuracion`. |
|------|------|------|------|
|------|------|------|------|
|------|------|------|------|
|------|------|------|------|
|------|------|------|------|
|------|------|------|------|
|------|------|------|------|
|------|------|------|------|
