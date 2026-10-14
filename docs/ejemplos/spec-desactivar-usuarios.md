# 0007 — Desactivar usuarios

> Ejemplo ilustrativo de una spec completa. No corresponde a un cambio real de este repositorio.

- **Estado:** Aprobada
- **Fecha:** 2026-10-09
- **Exploración:** `docs/exploraciones/0007-desactivar-usuarios.md`

## Objetivo

Permitir cortar el acceso de un usuario sin borrar sus datos, para conservar el historial de órdenes y la trazabilidad de auditoría.

**Historia**
Como administrador, quiero desactivar un usuario, para cortar el acceso de quien deja la empresa sin perder su historial.

## Alcance

**Incluye**
- Endpoint para desactivar un usuario.
- Bloqueo del login y de las sesiones activas del usuario desactivado.
- Registro de auditoría de la desactivación.

**No incluye**
- Reactivar usuarios.
- Desactivación masiva.
- Cambios en la interfaz web.
- Borrado de datos personales.

## Criterios de aceptación

### Comportamiento principal

#### CA-1: Un administrador desactiva un usuario
- **Dado** un usuario activo y un administrador autenticado
- **Cuando** el administrador envía `POST /users/{id}/deactivate`
- **Entonces** la respuesta es `200` y el usuario queda con estado `inactive`

#### CA-2: El usuario desactivado no puede iniciar sesión
- **Dado** un usuario desactivado con credenciales válidas
- **Cuando** intenta iniciar sesión
- **Entonces** la respuesta es `403` con el código de error `USER_INACTIVE`

#### CA-3: Las sesiones activas dejan de ser válidas
- **Dado** un usuario con un token de acceso vigente
- **Cuando** es desactivado y luego usa ese token
- **Entonces** la respuesta es `401`

#### CA-4: La desactivación queda auditada
- **Dado** un administrador que desactiva un usuario
- **Cuando** la operación termina con éxito
- **Entonces** existe un registro de auditoría con el administrador, el usuario afectado, la acción `user.deactivated` y la fecha

### Errores y casos borde

#### CA-5: Solo un administrador puede desactivar
- **Dado** un usuario autenticado sin rol de administrador
- **Cuando** envía `POST /users/{id}/deactivate`
- **Entonces** la respuesta es `403` y el usuario objetivo sigue activo

#### CA-6: Usuario inexistente
- **Dado** un ID que no corresponde a ningún usuario
- **Cuando** un administrador intenta desactivarlo
- **Entonces** la respuesta es `404`

#### CA-7: Desactivar un usuario ya inactivo es idempotente
- **Dado** un usuario ya desactivado
- **Cuando** un administrador lo desactiva otra vez
- **Entonces** la respuesta es `200`, el estado sigue `inactive` y no se crea un registro de auditoría nuevo

### Regresión

#### CA-8: El historial de órdenes sigue disponible
- **Dado** un usuario desactivado con órdenes previas
- **Cuando** un administrador consulta `GET /orders?user_id={id}`
- **Entonces** las órdenes se devuelven igual que antes de la desactivación

## Archivos permitidos

| Archivo | Acción | Criterios |
|---------|--------|-----------|
| `app/domain/user.py` | modificar | CA-1, CA-7 |
| `app/application/deactivate_user.py` | crear | CA-1, CA-4, CA-5, CA-6, CA-7 |
| `app/api/routes/users.py` | modificar | CA-1, CA-5, CA-6 |
| `app/application/login.py` | modificar | CA-2 |
| `app/api/dependencies/auth.py` | modificar | CA-3 |
| `migrations/0012_add_user_status.py` | crear | CA-1 |
| `tests/application/test_deactivate_user.py` | crear | CA-1, CA-4, CA-5, CA-6, CA-7 |
| `tests/api/test_users_deactivate.py` | crear | CA-1, CA-3, CA-5, CA-6 |
| `tests/application/test_login.py` | modificar | CA-2 |
| `tests/api/test_orders.py` | modificar | CA-8 |

## Notas

- **Decidido:** las sesiones activas se invalidan comprobando el estado del usuario en cada request autenticado (respuesta a la duda bloqueante planteada antes de escribir la spec).
- **Sin criterio de regresión:** el reporte mensual de ventas (`app/reports/sales.py`) lee usuarios por ID sin filtrar por estado; el cambio no lo afecta.
- **Duda no bloqueante:** texto exacto del mensaje de error de `USER_INACTIVE`.
