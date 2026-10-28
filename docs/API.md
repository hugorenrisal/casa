# Referencia de la API

API REST en JSON servida bajo `/api`. Todas las respuestas son `application/json` salvo
`GET /api/backup`, que se descarga como archivo.

- [Convenciones](#convenciones)
- [Autenticación y perfiles](#autenticación-y-perfiles)
- [Errores](#errores)
- [Salud](#salud)
- [Estado](#estado)
- [Copias](#copias)
- [Avisos push](#avisos-push)
- [Otros](#otros)

---

## Convenciones

| | |
|---|---|
| URL base | `https://<host>/api` |
| Formato | JSON UTF-8; cuerpo máximo de 4 MB |
| Concurrencia | Versión optimista en el campo `_version` del estado |
| Fechas | Claves en la zona horaria de la casa: día `YYYY-MM-DD`, semana `YYYY-Www`, mes `YYYY-MM` |

## Autenticación y perfiles

Dos cabeceras independientes, con propósitos distintos:

| Cabecera | Obligatoria | Propósito |
|---|---|---|
| `X-Casa` | solo si el servidor tiene `CODIGO_CASA` | **Acceso**: ¿perteneces a esta casa? Sin ella, `401` |
| `X-Perfil` | no (por defecto, perfil de padres) | **Autorización de producto**: qué reglas se aplican a la petición |

`X-Perfil` toma el `id` de uno de los perfiles definidos en
[`public/perfiles.js`](../public/perfiles.js): tres perfiles con rol `child` y `papas` con rol
`parent`.

> `X-Perfil` no es una credencial: dentro de la casa cualquiera puede elegir cualquier perfil.
> Sirve para hacer cumplir reglas de producto (un hijo no puede aprobarse sus tareas), no para
> autenticar. El control de acceso es `X-Casa`. Ver [SECURITY.md](../SECURITY.md).

**Límite de intentos:** 30 códigos de casa erróneos desde una IP en 15 minutos bloquean esa IP
durante el resto de la ventana (`429`). La comparación es de tiempo constante.

## Errores

Formato único:

```json
{ "error": "codigo_en_snake_case" }
```

| Estado | `error` | Cuándo |
|---|---|---|
| 400 | `estado_invalido` · `copia_invalida` · `suscripcion_invalida` | Cuerpo sin la forma mínima |
| 400 | `json_invalido` | JSON mal formado |
| 401 | `codigo_casa` | Falta `X-Casa` o no es correcto |
| 403 | `solo_papas` | Ruta reservada al perfil de padres |
| 403 | `cambio_no_permitido` | Un hijo intenta un cambio fuera de sus permisos (incluye `detalles`) |
| 404 | `no_encontrado` · `copia_no_encontrada` · `no_suscrito` | Recurso inexistente |
| 409 | `version_desfasada` | Otra escritura ocurrió antes (incluye `version` actual) |
| 409 | `ciclo_desfasado` | El estado enviado pertenece a una semana o un mes anterior |
| 413 | `peticion_invalida` | Cuerpo de más de 4 MB |
| 429 | `demasiados_intentos` | Límite de intentos del código de casa |
| 500 | `error_servidor` | Error interno (sin detalles en la respuesta) |

---

## Salud

### `GET /api/health`

Público, sin cabeceras. Lo usa la plataforma de despliegue como *health check*.

```json
{ "ok": true, "db": true, "protegida": false }
```

| Campo | Significado |
|---|---|
| `db` | Hay `DATABASE_URL` configurada |
| `protegida` | El servidor exige `X-Casa` |

## Estado

### `GET /api/state`

Devuelve el documento completo de la casa **ya al día**: con los ciclos de semana y mes aplicados,
la forma normalizada y las rachas recalculadas. Ver [MODELO-DE-DATOS.md](MODELO-DE-DATOS.md).

```json
{
  "members": [ { "id": "papas", "name": "…", "role": "parent", "color": "#8b6fd6", "load": "normal" } ],
  "fixedTasks": [ … ],
  "extraTasks": [ … ],
  "currentWeek": "2026-W44",
  "currentMonth": "2026-10",
  "_version": 128
}
```

### `PUT /api/state`

Guarda el documento completo. El cliente envía el `_version` que recibió.

| Respuesta | Cuerpo |
|---|---|
| `200` | `{ "ok": true, "version": 129 }` |
| `409` | `{ "error": "version_desfasada", "version": 130 }` → recargar y reaplicar |
| `403` | `{ "error": "cambio_no_permitido", "detalles": [ "…" ] }` |

Reglas aplicadas por el servidor, en este orden:

1. Comprobación de versión (`_version` = 0 se acepta como escritura sin control, para clientes
   antiguos).
2. Saneado completo por lista blanca: tipos, longitudes, enumerados y referencias.
3. Límites del perfil de hijo: solo puede marcar o entregar **sus** tareas, pedir premios y operar
   en el mercado. Economía, catálogos, calendario, rachas e historial son de los padres.
4. Pertenencia al ciclo actual.
5. Normalización, recálculo de rachas y escritura condicional.
6. Tras responder, avisos push a los afectados.

### `GET /api/backup` · padres

Descarga el estado como `casa-copia-YYYY-MM-DD.json` (`Content-Disposition: attachment`).

### `POST /api/restore` · padres

Sustituye el estado por el documento del cuerpo. **Antes guarda una copia preventiva** del estado
actual. Responde `{ "ok": true }`.

### `POST /api/reset` · padres

Vuelve al estado de ejemplo. **Antes guarda una copia preventiva.** Responde `{ "ok": true }`.

## Copias

### `GET /api/copias` · padres

Las 50 copias más recientes.

```json
{
  "copias": [
    { "id": 41, "dia": "2026-10-28", "motivo": "diaria", "version": 128,
      "creada": "2026-10-28T06:12:09.114Z", "bytes": 18432 }
  ]
}
```

`motivo`: `diaria` · `antes_de_reiniciar` · `antes_de_restaurar`.

### `POST /api/copias/:id/restaurar` · padres

Restaura la copia indicada tras guardar una copia preventiva. `404 copia_no_encontrada` si no
existe.

## Avisos push

### `GET /api/push/clave`

```json
{ "clave": "<clave pública VAPID en base64url>" }
```

### `POST /api/push/suscribir`

Registra el dispositivo para recibir los avisos del perfil indicado en `X-Perfil`.

```json
{ "suscripcion": { "endpoint": "https://web.push.apple.com/…", "keys": { "p256dh": "…", "auth": "…" } } }
```

Solo se aceptan `endpoint` HTTPS de servicios de push conocidos (Apple, Google, Mozilla,
Microsoft). Respuesta: `{ "ok": true, "perfil": "<id>" }`.

### `POST /api/push/baja`

```json
{ "endpoint": "https://…" }
```

### `POST /api/push/prueba`

Envía un aviso de prueba **solo** al `endpoint` indicado. `404 no_suscrito` si no está registrado.

### `GET /api/push/estado` · padres

Número de dispositivos suscritos por perfil.

```json
{ "suscripciones": { "papas": 2 } }
```

### `POST /api/push/recordatorios` · padres

Envía el recordatorio de la tarde. Pensado para un planificador externo (GitHub Actions).

| Parámetro | Efecto |
|---|---|
| — | Solo actúa entre 17:00 y 21:30 (hora de la casa) y **una vez al día** |
| `?simular=1` | Devuelve los avisos que enviaría, sin enviar ni registrar nada |
| `?forzar=1` | Ignora la ventana horaria (pruebas manuales) |

| Respuesta | Significado |
|---|---|
| `{ "ok": true, "dia": "…", "avisos": 3, "entregados": 4 }` | Enviado |
| `{ "ok": true, "yaEnviado": true, "dia": "…" }` | Ya se envió hoy |
| `{ "ok": true, "fueraDeHora": true, "hora": "22:31", "ventana": "17:00–21:30" }` | Fuera de la ventana |
| `{ "simulado": true, "dia": "…", "avisos": [ … ] }` | Simulación |

## Otros

| Ruta | Descripción |
|---|---|
| `GET /demo` | La app completa con datos ficticios; no realiza peticiones a la API |
| `GET /.well-known/assetlinks.json` | Digital Asset Links para la app de Android (requiere `ANDROID_FINGERPRINT`; si falta, `503`) |
| `GET /privacidad.html` | Política de privacidad |

## Ejemplo

```bash
curl -s https://<host>/api/state \
  -H "X-Casa: XXXX-XXXX-XXXX" \
  -H "X-Perfil: papas"
```
