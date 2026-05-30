# Plan de Pruebas — Hotel Nova

Documento con tablas para visualizar pruebas: Clases de Equivalencia, Valores Límite, Casos Unitarios y Casos de Integración.

---

## 1. Clases de Equivalencia

| Entrada | Clases válidas | Clases inválidas |
|---------|----------------|------------------|
| Email (registro / login / perfil) | 1. Formato válido: usuario@dominio.com (ej: usuario@hotel.com)<br>2. Permite mayúsculas/minúsculas y espacios en los extremos (se trimmean) | 3. Sin arroba ni dominio válido (ej: usuariohotel.com)<br>4. Múltiples arrobas (ej: usuario@@hotel)<br>5. Dominio inválido (ej: user@.com)<br>6. Vacío o solo espacios |
| DNI (registro / perfil) | 7. Cadena de exactamente 8 dígitos (ej: 12345678) | 8. Menos de 8 dígitos (ej: 1234567)<br>9. Más de 8 dígitos (ej: 123456789)<br>10. Contiene letras o caracteres especiales (ej: ABC12345)<br>11. Vacío |
| Contraseña (registro / cambio) | 12. Longitud entre 7 y 12 caracteres (ej: Secreto1)<br>13. Debe incluir al menos una mayúscula, una minúscula y un número<br>14. Solo permite letras y números<br>Nota: El campo de contraseña en frontend está enmascarado (asteriscos). El frontend muestra un patrón que permite algunos símbolos (! @ . - _), pero el servidor **rechaza** signos: `No se permiten signos. Solo se permiten letras y números`. | 15. Menor a 7 caracteres (ej: Secreto1)<br>16. Mayor a 12 caracteres (ej: Secreto1234567)<br>17. Sin mayúscula<br>18. Sin minúscula<br>19. Sin número<br>20. Con caracteres no permitidos (ej: #, ", :, $) |
| Nº de huéspedes (crear reserva) | 17. Entero entre 1 y 10 (ej: 1,2,3) | 18. Cero o negativo (ej: 0, -1)<br>19. Decimal o fraccionario (ej: 2.5)<br>20. Alfanumérico (ej: "dos")<br>21. Vacío |
| Nº de noches (crear reserva) | 22. Entero entre 1 y 365 (ej: 1, 7, 30) | 23. Cero o negativo (ej: 0)<br>24. Mayor a 365 (ej: 366)<br>25. Decimal (ej: 1.5)<br>26. Vacío |
| Método de pago / identificación (pagos) | 27. Valores soportados: "mercadopago" (ej: mercadopago) | 28. Método no soportado (ej: "efectivo")<br>29. Valor vacío o nulo<br>30. Formato inesperado (ej: objeto en lugar de cadena) |

---

## 2. Valores Límite

| Entrada | Clases válidas | Clases inválidas |
|---------|----------------|------------------|
| DNI | 31. Cadena de exactamente 8 dígitos (ej: 12345678) | 32. Menos de 8 dígitos (ej: 1234567)<br>33. Más de 8 dígitos (ej: 123456789) |
| Correo electrónico | 34. Correo válido @gmail.com (ej: usuario@gmail.com) | 35. Correo sin @ (ej: usuariogmail.com)<br>36. Correo con dominio distinto a gmail (ej: usuario@hotmail.com)<br>37. Vacío |
| Contraseña | 38. Longitud mínima 7 caracteres (ej: Secreto1)<br>39. Longitud máxima 12 caracteres (ej: Secreto12345)<br>40. Debe incluir al menos una mayúscula, una minúscula y un número<br>41. Solo permite letras y números<br>Nota: Campo en frontend enmascarado; cliente puede mostrar símbolos pero el servidor los rechazará. | 42. Menor a 7 caracteres (ej: Secre1)<br>43. Mayor a 12 caracteres (ej: Secreto1234567)<br>44. Sin mayúscula<br>45. Sin minúscula<br>46. Sin número<br>47. Caracteres no permitidos (ej: #, ", :, $) |
| Nombre del huésped | 46. 1 carácter (ej: A)<br>47. 50 caracteres o menos (ej: Juan Carlos Pérez) | 48. Vacío<br>49. Más de 50 caracteres |
| Número de habitación | 50. Valor entero positivo (ej: 101, 205, 310) | 51. Cero o negativo<br>52. Texto o alfanumérico inválido |
| Cantidad de noches | 53. Valor mínimo 1 noche<br>54. Valores intermedios válidos (ej: 2, 7, 30) | 55. Cero o negativo<br>56. Mayor al límite permitido por negocio |
| Total de reserva / pago | 57. Valor decimal positivo válido (ej: 350.00, 1899.99) | 58. Cero o negativo<br>59. Formato no numérico |

---

## 3. Casos de Prueba Unitarios

Formato: ID | Módulo / Función | Objetivo | Precondición | Pasos | Datos de entrada | Resultado esperado

| ID | Módulo / Función | Objetivo | Precondición | Pasos | Datos de entrada | Resultado esperado |
|----|------------------|---------|--------------|-------|------------------|-------------------:|
| UT-01 | `app.core.security.hash_password()` | Verificar que el hash no coincide con la contraseña y `verify_password` valida | Ninguna | Llamar `hash_password`, luego `verify_password` | "Secreto123" | `verify_password` -> True |
| UT-02 | `app.crud.usuario.create_user()` | Crear usuario correctamente | DB en memoria | Invocar create_user con datos válidos | {dni,email,nombre,password} | Nuevo usuario persistido, password hasheado |
| UT-03 | `app.crud.reserva.create_reserva()` | Crear reserva con cálculos correctos | Habitacion disponible | Llamar función de creación | {habitacion_id,fecha_checkin,fecha_checkout,huéspedes} | Reserva con `total` calculado correctamente |
| UT-04 | `app.services.mercado_pago.create_reserva_preference()` | Generar preference con campos obligatorios | MP SDK mock | Llamar create_reserva_preference | reserva_id, total, payer_email | Retorna `init_point` y `id` |
| UT-05 | `app.services.email_service.send_reserva_confirmed_email()` | Encolar envío de email | SMTP mock | Llamar función | email, reserva_id | Llamada realizada al servicio (mock) |
| UT-06 | `app.crud.reserva._register_approved_payment()` | Evitar duplicados de pago por referencia | DB con pagos previos | Intentar registrar con misma referencia | reserva_id, payment_id | No duplica, retorna False si existe |

Notas:
- Usar `pytest` y fixtures para DB (`sqlite:///:memory:`) y mocks (`pytest-mock` o `unittest.mock`).
- Aislar dependencias externas (MercadoPago, SMTP) con mocks.

---

### Casos de Prueba Unitarios — DNI

| Entrada(CP) | C.Validas | C.Invalida | Acciones | Salida Esperada |
|--------------|-----------|------------|----------|-----------------|
| 12345678 | 31 |  | • Seleccionar campo DNI<br>• Registrar DNI<br>• Seleccionar siguiente campo | DNI aceptado |
| 1234567 |  | 32 | • Seleccionar campo DNI<br>• Registrar DNI<br>• Seleccionar siguiente campo | Error: El DNI debe tener exactamente 8 dígitos |
| 123456789 |  | 33 | • Seleccionar campo DNI<br>• Registrar DNI<br>• Seleccionar siguiente campo | Error: El DNI debe tener exactamente 8 dígitos |
| ABC12345 |  | 33 | • Seleccionar campo DNI<br>• Registrar DNI<br>• Seleccionar siguiente campo | Error: Solo se aceptan números |
| "" |  | 32, 33 | • Seleccionar campo DNI<br>• Registrar DNI<br>• Seleccionar siguiente campo | Error: Campo obligatorio |

### Casos de Prueba Unitarios — Correo electrónico

| Entrada(CP) | C.Validas | C.Invalida | Acciones | Salida Esperada |
|--------------|-----------|------------|----------|-----------------|
| usuario@gmail.com | 34 |  | • Seleccionar campo Correo<br>• Registrar correo<br>• Seleccionar siguiente campo | Correo aceptado |
| usuario@hotmail.com |  | 35 | • Seleccionar campo Correo<br>• Registrar correo<br>• Seleccionar siguiente campo | Error: Solo se permiten correos @gmail.com |
| usuariogmail.com |  | 35 | • Seleccionar campo Correo<br>• Registrar correo<br>• Seleccionar siguiente campo | Error: Correo inválido |
| "" |  | 37 | • Seleccionar campo Correo<br>• Registrar correo<br>• Seleccionar siguiente campo | Error: Campo obligatorio |

### Casos de Prueba Unitarios — Contraseña

| Entrada(CP) | C.Validas | C.Invalida | Acciones | Salida Esperada |
|--------------|-----------|------------|----------|-----------------|
| Secreto1 | 38, 40, 41 |  | • Seleccionar campo Contraseña<br>• Registrar contraseña (campo enmascarado)<br>• Seleccionar siguiente campo | Contraseña aceptada |
| Secreto12345 | 39, 40, 41 |  | • Seleccionar campo Contraseña<br>• Registrar contraseña (campo enmascarado)<br>• Seleccionar siguiente campo | Contraseña aceptada |
| Secre1 |  | 42 | • Seleccionar campo Contraseña<br>• Registrar contraseña (campo enmascarado)<br>• Seleccionar siguiente campo | Error: La contraseña debe tener entre 7 y 12 caracteres |
| SECRETO1 |  | 44 | • Seleccionar campo Contraseña<br>• Registrar contraseña (campo enmascarado)<br>• Seleccionar siguiente campo | Error: Debe incluir minúsculas |
| secreto1 |  | 45 | • Seleccionar campo Contraseña<br>• Registrar contraseña (campo enmascarado)<br>• Seleccionar siguiente campo | Error: Debe incluir mayúsculas |
| Secreto |  | 46 | • Seleccionar campo Contraseña<br>• Registrar contraseña (campo enmascarado)<br>• Seleccionar siguiente campo | Error: Debe incluir al menos un número |
| Secreto#1 |  | 47 | • Seleccionar campo Contraseña<br>• Registrar contraseña (campo enmascarado)<br>• Seleccionar siguiente campo | Error: No se permiten signos. Solo se permiten letras y números |

### Casos de Prueba Unitarios — Nombre del huésped

| Entrada(CP) | C.Validas | C.Invalida | Acciones | Salida Esperada |
|--------------|-----------|------------|----------|-----------------|
| Juan | 46, 47 |  | • Seleccionar campo Nombre<br>• Registrar nombre<br>• Seleccionar siguiente campo | Nombre aceptado |
| Juan Carlos Pérez | 47 |  | • Seleccionar campo Nombre<br>• Registrar nombre<br>• Seleccionar siguiente campo | Nombre aceptado |
| NombreMuyLargoQueSuperaElLimitePermitido |  | 49 | • Seleccionar campo Nombre<br>• Registrar nombre<br>• Seleccionar siguiente campo | Error: El nombre no puede superar el límite permitido |
| "" |  | 48 | • Seleccionar campo Nombre<br>• Registrar nombre<br>• Seleccionar siguiente campo | Error: Campo obligatorio |


## 4. Casos de Prueba de Integración

Formato: ID | Flujo | Objetivo | Precondición | Pasos | Resultado esperado

| ID | Flujo | Objetivo | Precondición | Pasos | Resultado esperado |
|----|-------|---------|--------------|-------|-------------------:|
| IT-01 | Registro -> Login -> `GET /auth/me` | Verificar flujo completo de registro y obtención de perfil | DB limpia | 1) POST /auth/register 2) POST /auth/login 3) GET /auth/me (Authorization: Bearer) | 201 -> 200 + token -> 200 y datos del usuario |
| IT-02 | Crear reserva -> Generar preferencia MP -> Simular pago (webhook) | Validar integración reserva-pago-notificación | Usuario autenticado, habitación disponible | 1) POST /reservas 2) POST /reservas/{id}/pagar (obtener init_point) 3) Simular webhook POST /reservas/webhook/mercadopago con payment aprobado y external_reference=reserva_id | Reserva cambia a `activo`, pago registrado, correo encola/do |
| IT-03 | Pago fallido -> Reserva queda `pendiente` y no se registra pago | Manejo de errores en flujo de pagos | Usuario autenticado, habitación disponible | 1) POST /reservas 2) Simular webhook con status != "approved" | Reserva `pendiente`, no hay pago aprobado en DB |
| IT-04 | Obtener historial paginado (`GET /reservas?limit=10&page=2`) | Validar paginación y filtros | Usuario con >20 reservas | 1) Crear >20 reservas 2) GET /reservas?limit=10&skip=10 | 200 y 10 ítems retornados |
| IT-05 | Admin: crear pago manual y obtener resumen mensual | Validar endpoints admin para pagos | Usuario admin + reservas/pagos en DB | 1) POST /pagos 2) GET /pagos/resumen/mes-actual | Pago creado y resumen con totales correctos |

Notas de Integración:
- Ejecutar en entorno de staging o con servicios mockeados (MercadoPago, SMTP) cuando no se disponga de claves reales.
- Probar webhooks con payloads reales (simulados) y verificar idempotencia.

---

### Caso de Prueba de Integración - Módulo Registro de Usuario y Reserva

| id | Entrada (DNI, Email, Contraseña, Nombre, Reserva) | C.Válidas | C.Inválidas | Acciones | Salida |
|----|---------------------------------------------------|-----------:|------------:|---------|--------|
| 1 | 12345678, usuario@gmail.com, Secreto1, Juan Pérez, - | 31,34,38,40,41,46 |  | 1) POST /auth/register con datos válidos<br>2) Verificar 201 y DB | Usuario registrado correctamente (201) |
| 2 | 12345679, huesped@gmail.com, Secreto12345, Ana López, - | 31,34,39,40,41,47 |  | 1) POST /auth/register<br>2) Verificar respuesta y DB | Usuario registrado correctamente (201) |
| 3 | 12345670, cliente@gmail.com, Secreto1234, Carlos Ruiz, Hab:101, Noches:2, Total:350.00 | 31,34,39,40,41,47,50,53,57 |  | 1) POST /auth/register 2) POST /reservas (autenticado) 3) Verificar reserva creada con total calculado | Usuario y reserva registrados correctamente; total correcto |
| 4 | 1234567, usuario@gmail.com, Secreto1, Juan Pérez, - |  | 32 | 1) POST /auth/register con DNI inválido | 400 + Error: DNI inválido |
| 5 | 12345678, usuario@hotmail.com, Secreto1, Juan Pérez, - |  | 35 | 1) POST /auth/register con email @hotmail.com | 400 + Error: Solo se permiten correos @gmail.com |
| 6 | 12345678, usuario@gmail.com, Secre1, Juan Pérez, - |  | 42 | 1) POST /auth/register con contraseña corta | 400 + Error: La contraseña debe tener entre 7 y 12 caracteres |
| 7 | 12345678, usuario@gmail.com, Secreto#1, Juan Pérez, - |  | 47 | 1) POST /auth/register con signos en contraseña | 400 + Error: No se permiten signos. Solo se permiten letras y números |
| 8 | 12345678, usuario@gmail.com, Secreto1, Juan Pérez, Hab:101, Noches:0 |  | 55 | 1) POST /reservas con 0 noches | 400 + Error: La cantidad de noches debe ser mayor a 0 |
| 9 | 12345678, usuario@gmail.com, Secreto1, Juan Pérez, - (duplicado) |  | 32,35 | 1) POST /auth/register con DNI/email ya existente | 409 + Error: Usuario ya existe |
| 10 | "", "", "", "", - |  | 5,9,19,25,28 | 1) POST /auth/register con campos vacíos | 400 + Error: Campos obligatorios |
