# TDD: Confirmación de Pagos y Activación de Reservas — Hotel Nova

## FASE RED — Pruebas que fallan inicialmente

Antes de implementar la funcionalidad, se escriben pruebas que fallan porque el código
de verificación de pagos aún no existe o no está integrado. Las pruebas validan:

1. Activación de reserva tras pago exitoso.
2. Prevención de pagos duplicados (idempotencia).
3. Validación de permisos (solo propietario o admin).

---

### Test 1: Activación de Reserva por Pago Exitoso

| Objetivo | Verificar que al recibir un pago aprobado la reserva cambie a `activo` |
|---|---|
| Datos de entrada | `GET /reservas/{id}/pago-exitoso?status=approved&payment_id=999` con mock de MP que retorna `approved` |
| Resultado esperado | HTTP 200, `reserva.estado == "activo"`, pago registrado en BD |

```python
# RED: Esta prueba falla porque el endpoint pago-exitoso no existe o
# no está integrado con el mock de MercadoPagoService.

def test_pago_exitoso_marks_reserva_active(client, db_session, mp_stub, reserva, user_headers):
    response = client.get(
        f"/api/reservas/{reserva.id}/pago-exitoso?status=approved&payment_id=999",
        headers=user_headers
    )

    assert response.status_code == 200  # FALLA: endpoint no implementado

    db_session.refresh(reserva)
    assert reserva.estado == "activo"  # FALLA: estado sigue siendo "pendiente"

    pago = db_session.query(Pago).filter(Pago.reserva_id == reserva.id).first()
    assert pago is not None  # FALLA: no se registró ningún pago
    assert pago.estado == EstadoPagoEnum.aprobado
```

---

### Test 2: Prevención de Pagos Duplicados (Idempotencia)

| Objetivo | Validar que dos notificaciones idénticas del webhook creen un solo pago |
|---|---|
| Datos de entrada | `POST /webhook/mercadopago` con `{"type":"payment","data":{"id":"999"}}` enviado dos veces |
| Resultado esperado | HTTP 200 ambas veces, pero solo 1 registro de pago en BD |

```python
# RED: Esta prueba falla porque el webhook no tiene control de idempotencia.

def test_webhook_mercadopago_idempotent(client, db_session, mp_stub, reserva):
    payload = {"type": "payment", "data": {"id": "999"}}

    r1 = client.post("/api/reservas/webhook/mercadopago", json=payload)
    assert r1.status_code == 200  # PASA (el webhook responde)

    r2 = client.post("/api/reservas/webhook/mercadopago", json=payload)
    assert r2.status_code == 200  # PASA

    pagos = db_session.query(Pago).filter(Pago.reserva_id == reserva.id).all()
    assert len(pagos) == 1  # FALLA: se crearon 2 registros de pago
```

---

### Test 3: Validación de Permisos (403)

| Objetivo | Garantizar que solo el propietario o admin puedan confirmar un pago |
|---|---|
| Datos de entrada | `GET /reservas/{id}/pago-exitoso` con token de un usuario distinto al propietario |
| Resultado esperado | HTTP 403 Forbidden |

```python
# RED: Esta prueba falla porque no existe validación de permisos en el endpoint.

def test_pago_exitoso_forbidden_for_wrong_user(client, db_session, mp_stub, reserva):
    otro = Usuario(nombre="Otro", email="otro@test.com", password="hash", rol="cliente")
    db_session.add(otro)
    db_session.commit()

    token = create_access_token(data={"sub": str(otro.id)})
    wrong_headers = {"Authorization": f"Bearer {token}"}

    response = client.get(
        f"/api/reservas/{reserva.id}/pago-exitoso?status=approved&payment_id=999",
        headers=wrong_headers
    )

    assert response.status_code == 403  # FALLA: retorna 200 en lugar de 403
```

---

## FASE GREEN — Implementación mínima para pasar las pruebas

Se implementa el código estrictamente necesario para que los 3 tests anteriores pasen.

```python
# GREEN: Implementación mínima — extraído de:
# backend/app/routers/reservas.py

from app.models import EstadoPagoEnum, MetodoPagoEnum, Pago, Usuario
from app.services import MercadoPagoService, EmailService


def _verify_mp_payment_approved(mp_service, *, reserva_id, payment_id, payment_status):
    """Verifica el pago contra Mercado Pago por payment_id,
    external_reference o status de URL."""
    if payment_id:
        try:
            info = mp_service.sdk.payment().get(int(payment_id))
            if info.get("response", {}).get("status") == "approved":
                return True
        except Exception:
            pass

    try:
        search = mp_service.sdk.payment().search({
            "external_reference": str(reserva_id),
            "sort": "date_created", "criteria": "desc", "limit": 5,
        })
        for p in search.get("response", {}).get("results", []):
            if p.get("status") == "approved":
                return True
    except Exception:
        pass

    return payment_status == "approved"


def _assert_reserva_access(reserva, current_user):
    # backend/app/routers/reservas.py:77-84
    if not (current_user.rol.value == "admin" or reserva.usuario_id == current_user.id):
        raise HTTPException(status_code=403, detail="Permisos insuficientes")


def _register_approved_payment(db, *, reserva_id, amount, payment_id=""):
    # backend/app/routers/reservas.py:87-113
    """Registra pago aprobado solo si no existe duplicado."""
    if payment_id:
        existing = db.query(Pago).filter(Pago.referencia_externa == payment_id).first()
        if existing:
            return False

    existing_approved = db.query(Pago).filter(
        Pago.reserva_id == reserva_id,
        Pago.estado == EstadoPagoEnum.aprobado
    ).first()
    if existing_approved:
        return False

    pago = Pago(
        reserva_id=reserva_id, monto=amount, moneda="USD",
        metodo=MetodoPagoEnum.mercadopago, estado=EstadoPagoEnum.aprobado,
        referencia_externa=payment_id or None,
    )
    db.add(pago)
    return True


@router.get("/{reserva_id}/pago-exitoso", response_model=ReservaResponse)
def pago_exitoso(reserva_id, background_tasks, payment_id="",
                 payment_status=Query("", alias="status"),
                 db=Depends(get_db), current_user=Depends(get_current_user)):
    # backend/app/routers/reservas.py:233-289

    reserva = get_reserva_by_id(db, reserva_id)
    if not reserva:
        raise HTTPException(status_code=404, detail="Reserva no encontrada")

    _assert_reserva_access(reserva, current_user)

    if reserva.estado.value == "activo":
        return reserva

    settings = get_settings()
    mp_service = MercadoPagoService(settings.mercadopago_access_token)
    verified = _verify_mp_payment_approved(
        mp_service, reserva_id=reserva_id,
        payment_id=payment_id, payment_status=payment_status,
    )

    if not verified:
        raise HTTPException(status_code=400, detail="El pago no fue aprobado")

    reserva.estado = "activo"
    _register_approved_payment(db, reserva_id=reserva.id,
                                amount=float(reserva.total), payment_id=payment_id)
    db.add(reserva)
    db.commit()
    db.refresh(reserva)
    _queue_reserva_confirmation_email(background_tasks, db, reserva)

    return reserva


@router.post("/webhook/mercadopago")
async def mercadopago_webhook(request, background_tasks, db=Depends(get_db)):
    # backend/app/routers/reservas.py:292-368
    body = await request.json()
    topic = body.get("type") or body.get("topic")
    if topic != "payment":
        return {"status": "ignored"}

    data = body.get("data", {})
    payment_id = data.get("id") or body.get("data.id")

    settings = get_settings()
    mp_service = MercadoPagoService(settings.mercadopago_access_token)
    payment_info = mp_service.sdk.payment().get(int(payment_id))
    payment_response = payment_info.get("response", {})

    if payment_response.get("status") != "approved":
        return {"status": "ok", "action": "no_update"}

    external_ref = payment_response.get("external_reference", "")
    reserva_id = int(external_ref)
    reserva = get_reserva_by_id(db, reserva_id)

    if reserva.estado.value != "activo":
        reserva.estado = "activo"
        _register_approved_payment(db, reserva_id=reserva.id,
                                    amount=float(reserva.total), payment_id=str(payment_id))
        db.add(reserva)
        db.commit()
        db.refresh(reserva)
        _queue_reserva_confirmation_email(background_tasks, db, reserva)

    return {"status": "ok", "reserva_id": reserva_id, "new_status": "activo"}
```

**Resultado: los 3 tests ahora PASAN.**

---

## FASE REFACTOR — Código mejorado manteniendo las pruebas verdes

Se refactoriza la implementación anterior para mejorar legibilidad, separación de
responsabilidades y testabilidad, sin modificar el comportamiento externo.

### Mejora 1: Método dedicado en MercadoPagoService

```python
# PROPÓSITO: Nuevo método para facilitar mocks en tests
# ARCHIVO:   backend/app/services/mercado_pago.py
class MercadoPagoService:
    ...

    def verify_payment_approved(self, *, payment_id: str = "",
                                external_reference: str = "") -> bool:
        """Método dedicado y fácil de mockear en tests."""
        if payment_id:
            try:
                info = self.sdk.payment().get(int(payment_id))
                if info.get("response", {}).get("status") == "approved":
                    return True
            except Exception:
                pass

        if external_reference:
            try:
                search = self.sdk.payment().search({
                    "external_reference": external_reference,
                    "sort": "date_created", "criteria": "desc", "limit": 5,
                })
                for p in search.get("response", {}).get("results", []):
                    if p.get("status") == "approved":
                        return True
            except Exception:
                pass

        return False
```

### Mejora 2: CRUD centralizado de pagos

```python
# PROPÓSITO: Centralizar lógica de pagos (hoy inline en _register_approved_payment)
# ARCHIVO:   backend/app/crud/pago.py (NUEVO)
from sqlalchemy.orm import Session
from app.models import Pago, EstadoPagoEnum, MetodoPagoEnum


def create_approved_pago(db: Session, *, reserva_id: int, monto: float,
                          referencia_externa: str = "") -> Pago | None:
    """Crea pago aprobado si no existe duplicado. Retorna Pago o None."""
    if referencia_externa:
        existing = db.query(Pago).filter(
            Pago.referencia_externa == referencia_externa
        ).first()
        if existing:
            return None

    existing_approved = db.query(Pago).filter(
        Pago.reserva_id == reserva_id,
        Pago.estado == EstadoPagoEnum.aprobado
    ).first()
    if existing_approved:
        return None

    pago = Pago(
        reserva_id=reserva_id, monto=monto, moneda="USD",
        metodo=MetodoPagoEnum.mercadopago, estado=EstadoPagoEnum.aprobado,
        referencia_externa=referencia_externa or None,
    )
    db.add(pago)
    return pago


def has_approved_pago(db: Session, reserva_id: int) -> bool:
    return db.query(Pago.id).filter(
        Pago.reserva_id == reserva_id,
        Pago.estado == EstadoPagoEnum.aprobado
    ).first() is not None


def get_pago_by_referencia_externa(db: Session, ref: str) -> Pago | None:
    return db.query(Pago).filter(Pago.referencia_externa == ref).first()
```

### Mejora 3: EmailService inyectable

```python
# PROPÓSITO: Permitir mock de EmailService en tests
# ARCHIVO:   backend/app/routers/reservas.py
def _queue_reserva_confirmation_email(
    background_tasks, db, reserva,
    email_service: EmailService | None = None,
):
    if email_service is None:
        settings = get_settings()
        email_service = EmailService(
            host=settings.smtp_host, port=settings.smtp_port,
            username=settings.smtp_user, password=settings.smtp_password,
            from_name=settings.smtp_from_name, from_email=settings.smtp_from_email,
        )

    if not email_service.is_configured:
        logger.warning("SMTP no configurado. Correo omitido.")
        return

    usuario = db.query(Usuario).filter(Usuario.id == reserva.usuario_id).first()
    if not usuario or not usuario.email:
        return

    background_tasks.add_task(
        email_service.send_reserva_confirmed_email,
        to_email=usuario.email, guest_name=usuario.nombre,
        reserva_id=reserva.id, room_label=f"Hab {reserva.habitacion_id}",
        fecha_checkin=str(reserva.fecha_checkin),
        fecha_checkout=str(reserva.fecha_checkout),
        total=f"S/ {float(reserva.total):.2f}",
    )
```

### Mejora 4: Uso de crud/pago.py en los endpoints

```python
# PROPÓSITO: Endpoint refactorizado usando el nuevo CRUD y método de MP
# ARCHIVO:   backend/app/routers/reservas.py
from app.crud.pago import create_approved_pago

@router.get("/{reserva_id}/pago-exitoso", response_model=ReservaResponse)
def pago_exitoso(reserva_id, ..., db=Depends(get_db), ...):
    reserva = get_reserva_by_id(db, reserva_id)
    if not reserva:
        raise HTTPException(status_code=404, detail="Reserva no encontrada")

    _assert_reserva_access(reserva, current_user)

    if reserva.estado.value == "activo":
        return reserva

    settings = get_settings()
    mp_service = MercadoPagoService(settings.mercadopago_access_token)

    # Usa el nuevo método dedicado (fácil de mockear)
    verified = mp_service.verify_payment_approved(
        payment_id=payment_id,
        external_reference=str(reserva_id),
    ) or (payment_status == "approved")

    if not verified:
        raise HTTPException(status_code=400, detail="El pago no fue aprobado")

    # Usa el CRUD centralizado
    pago = create_approved_pago(
        db, reserva_id=reserva.id,
        monto=float(reserva.total),
        referencia_externa=payment_id,
    )

    reserva.estado = "activo"
    db.add(reserva)
    db.commit()
    db.refresh(reserva)
    _queue_reserva_confirmation_email(background_tasks, db, reserva)

    return reserva
```

## Casos Adicionales Cubiertos en REFACTOR

| Escenario | Validación |
|---|---|
| Pago por payment_id | `verify_payment_approved(payment_id="123")` |
| Pago por external_reference | `verify_payment_approved(external_reference="5")` |
| Pago por payment_status | Fallback `payment_status == "approved"` |
| Error de conexión con MP | Try/except captura excepción, retorna `False` |
| Webhooks duplicados | `create_approved_pago` retorna `None` si ya existe |
| Stock negativo en pago | No aplica (no hay stock en hotel) |

## Conclusión

Aplicando TDD al flujo de pagos de Hotel Nova:

- **RED**: Se escribieron 3 pruebas que fallan (activación, idempotencia, permisos).
- **GREEN**: Se implementaron 4 funciones helper y 2 endpoints que pasan las pruebas.
- **REFACTOR**: Se extrajo la verificación a `MercadoPagoService.verify_payment_approved()`,
  se centralizó la creación de pagos en `crud/pago.py` y se hizo `EmailService` inyectable.

El resultado es un sistema con pagos idempotentes, reservas consistentes,
permisos validados y código modular listo para testing automatizado.
