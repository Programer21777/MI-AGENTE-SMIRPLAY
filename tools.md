# Tools

Este documento describe las *skills* (APIs) disponibles para **AgentSmirplay** mediante un backend en **Django REST Framework (DRF)**.
Cada skill incluye: endpoint, método, entradas, salidas y límites de seguridad.

> Base URL sugerida: `/api/v1/`  
> Formato: JSON  
> Auth: Token/JWT (según tu proyecto)

---

## auth_login
**Endpoint:** `POST /api/v1/auth/login/`  
**Recibe (body JSON):**
- `username` (string)
- `password` (string)

**Qué hace:** autentica al usuario y devuelve token de acceso para usar el resto de endpoints.  
**Devuelve (ejemplo):**
- `access` (string) y `refresh` (string) si usas JWT  
o
- `token` (string) si usas TokenAuth

**Límites de seguridad:**
- No registrar contraseñas en logs.
- Rate limit (ej. 5 intentos/min).
- Respuestas genéricas en error (no decir si usuario existe).

---

## get_profile
**Endpoint:** `GET /api/v1/users/me/`  
**Recibe:** nada (solo token en headers)  
**Qué hace:** devuelve el perfil del usuario autenticado.  
**Devuelve (ejemplo):**
- `id` (int)
- `name` (string)
- `timezone` (string)

**Límites de seguridad:**
- Solo доступ a `me` (no ver perfiles de otros sin permiso).
- No devolver datos sensibles innecesarios.

---

## reverse_geocode
**Endpoint:** `GET /api/v1/geo/reverse/`  
**Recibe (query params):**
- `lat` (float) → latitud (ej. `32.52`)
- `lon` (float) → longitud (ej. `-117.03`)

**Qué hace:** convierte coordenadas (`lat`, `lon`) en una ubicación aproximada (ciudad/estado/colonia) para mostrar en pantalla o registrar contexto.  
**Devuelve (ejemplo):**
- `lat`, `lon`
- `address` (string)
- `city` (string)
- `state` (string)
- `country` (string)

**Límites de seguridad:**
- No guardar ubicación precisa sin consentimiento.
- Si se almacena, redondear coordenadas (privacidad).
- Validar rangos: `lat` [-90, 90], `lon` [-180, 180].

---

## list_posts
**Endpoint:** `GET /api/v1/posts/`  
**Recibe (query params opcionales):**
- `q` (string) búsqueda
- `page` (int) paginación
- `type` (string) filtro (ej. `match`, `news`, `promo`)

**Qué hace:** lista publicaciones/entradas (por ejemplo para contenido de Smirplay).  
**Devuelve (ejemplo):**
- `count` (int)
- `results` (lista) con objetos que incluyen `id`, `title`, `type`, `created_at`

**Límites de seguridad:**
- Paginación obligatoria para evitar respuestas enormes.
- Permisos: solo leer lo permitido (público/propio).

---

## create_post
**Endpoint:** `POST /api/v1/posts/`  
**Recibe (body JSON):**
- `title` (string)
- `content` (string)
- `type` (string)
- `metadata` (object opcional)

**Qué hace:** crea una publicación nueva.  
**Devuelve (ejemplo):**
- `id` (int)
- `title`, `content`, `type`
- `created_at`

**Límites de seguridad:**
- Validar longitud máxima de texto.
- Sanitizar contenido si se renderiza en HTML.
- No permitir guardar secretos en `metadata`.

---

## update_post
**Endpoint:** `PATCH /api/v1/posts/{id}/`  
**Recibe:**
- `id` (path int)
- Body JSON parcial con campos a actualizar (ej. `title`, `content`, `type`, `metadata`)

**Qué hace:** actualiza una publicación existente.  
**Devuelve:** el objeto actualizado.

**Límites de seguridad:**
- Solo el dueño o un rol admin puede editar.
- Guardar auditoría: quién editó y cuándo.

---

## delete_post
**Endpoint:** `DELETE /api/v1/posts/{id}/`  
**Recibe:**
- `id` (path int)

**Qué hace:** elimina una publicación.  
**Devuelve:** `204 No Content`

**Límites de seguridad (IMPORTANTE):**
- **No eliminar sin confirmar**: requerir confirmación explícita (ej. `?confirm=true` o doble confirmación en UI).
- Preferir *soft delete* (`is_deleted=true`) para recuperación.
- Bloquear eliminación masiva por defecto.

---

## create_reminder
**Endpoint:** `POST /api/v1/reminders/`  
**Recibe (body JSON):**
- `title` (string)
- `cron` (string) ejemplo: `0 9 * * *`
- `timezone` (string) ejemplo: `UTC-6`
- `payload` (object opcional)

**Qué hace:** programa un recordatorio recurrente para el usuario.  
**Devuelve (ejemplo):**
- `id` (int)
- `next_run` (datetime)
- `enabled` (bool)

**Límites de seguridad:**
- Validar el cron (formato y frecuencia mínima).
- No permitir frecuencia extrema sin permiso (ej. cada minuto).
- Permitir desactivar (`enabled=false`) sin borrar.

---

## health_check
**Endpoint:** `GET /api/v1/health/`  
**Recibe:** nada  
**Qué hace:** verifica estado del servicio (API/DB).  
**Devuelve (ejemplo):**
- `status`: `"ok"`
- `db`: `"ok"`

**Límites de seguridad:**
- No exponer información interna sensible (versiones, rutas, credenciales).
