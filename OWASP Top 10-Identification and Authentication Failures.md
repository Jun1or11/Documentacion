# OWASP Top 10 — A07: Identification and Authentication Failures — Hotel Nova

## 1. Descripción técnica

Identification and Authentication Failures ocurre cuando el sistema no gestiona correctamente la identidad de los usuarios ni la seguridad del proceso de autenticación. Esto incluye contraseñas débiles o expuestas, falta de límite en intentos de inicio de sesión, tokens mal configurados, y credenciales transmitidas o almacenadas de forma insegura.

En Hotel Nova, estas fallas se manifiestan en la exposición visual de la contraseña durante el registro, la ausencia de rate limiting en el endpoint de login, y el uso del algoritmo HS256 para firmar los JWTs, lo que reduce la seguridad del mecanismo de autenticación.

## 2. Posible aparición en el sistema

Módulo afectado: Autenticación (`frontend/src/pages/Login.tsx`, `backend/app/routers/auth.py`, `backend/app/core/config.py`).

- **Contraseña visible durante el registro** — En `Login.tsx:270` el campo de contraseña usa `type="text"` en la pestaña de registro, mostrando la contraseña en texto plano en pantalla. Esto expone la credencial a observación visual (shoulder surfing) en entornos públicos.
- **Sin rate limiting en login** — El endpoint `POST /api/auth` en `auth.py:125-159` no implementa límite de intentos por IP o por usuario. Un atacante puede realizar cientos de intentos de contraseña por segundo (ataque de fuerza bruta).
- **JWT firmado con HS256** — En `config.py:19` se define `algorithm: str = "HS256"`. HS256 es un algoritmo simétrico que usa la misma clave (`secret_key`) para firmar y verificar. Si la clave se filtra, cualquier atacante puede forjar tokens válidos. Además, HS256 es menos recomendado que RS256 (asimétrico) para entornos con múltiples servicios.

| Componente | Archivo | Línea | Problema |
|---|---|---|---|
| Frontend | `frontend/src/pages/Login.tsx` | 270 | `type="text"` en registro expone contraseña |
| Backend | `backend/app/routers/auth.py` | 125-159 | Sin rate limiting en login |
| Backend | `backend/app/core/config.py` | 19 | Algoritmo HS256 (simétrico) |

Consecuencia para la empresa: cuentas de usuario comprometidas por fuerza bruta, suplantación de identidad, acceso no autorizado a datos de reservas y pagos, y posible robo de información sensible.

## 3. Medidas de mitigación

- **Rate limiting en login**: implementar un middleware de throttling (por IP y por usuario) en el endpoint `POST /api/auth`. Puede usarse `slowapi` con FastAPI para limitar a 5 intentos por minuto.
- **Ocultar contraseña en registro**: cambiar `type="text"` a `type="password"` en `Login.tsx:270` y agregar un botón de "mostrar/ocultar" si se desea dar visibilidad opcional.
- **Migrar a RS256 (asimétrico)**: generar un par de llaves RSA (`private.pem` / `public.pem`), usar la llave privada para firmar y la pública solo para verificar. Esto evita que una fuga del `secret_key` permita forjar tokens.
- **Registrar y auditar intentos fallidos de autenticación** en la base de datos o en logs centralizados para detectar patrones de ataque.

## 4. Diseño de pruebas de seguridad

Plantea una prueba manual o automatizada que permita detectar la vulnerabilidad.

Especifica:

- Objetivo de la prueba
- Procedimiento
- Resultado esperado
- Evidencia esperada

### Prueba 1: Exposición de contraseña en registro

| Campo | Detalle |
|---|---|
| Objetivo | Verificar que la contraseña no sea visible en texto plano durante el registro. |
| Procedimiento | Navegar a `/register`. Hacer clic en el campo de contraseña y escribir una contraseña. Observar si los caracteres se muestran en texto plano o enmascarados. |
| Resultado esperado | La contraseña debe mostrarse como `••••••` (enmascarada) y no en texto plano. |
| Evidencia esperada | Captura de pantalla del campo de contraseña en registro mostrando caracteres enmascarados. |

### Prueba 2: Rate limiting en login

| Campo | Detalle |
|---|---|
| Objetivo | Verificar que el endpoint de login bloquee intentos repetidos de autenticación. |
| Procedimiento | Enviar 10 solicitudes consecutivas a `POST /api/auth` con credenciales inválidas desde la misma IP en menos de 10 segundos. |
| Resultado esperado | Después del quinto intento, el servidor debe responder con `HTTP 429 Too Many Requests` y bloquear temporalmente la IP. |
| Evidencia esperada | Logs del servidor mostrando los intentos bloqueados y captura de la respuesta HTTP 429. |

### Prueba 3: Algoritmo de firma JWT

| Campo | Detalle |
|---|---|
| Objetivo | Verificar que el JWT se firme con un algoritmo seguro (idealmente RS256). |
| Procedimiento | Iniciar sesión, capturar el JWT del `access_token` y decodificarlo (sin verificar) en [jwt.io](https://jwt.io) para inspeccionar el header `alg`. |
| Resultado esperado | El header `alg` debe indicar `RS256` (o al menos `HS256` con una clave robusta de 256+ bits). |
| Evidencia esperada | Captura de pantalla del payload decodificado mostrando el algoritmo en el header del JWT. |
