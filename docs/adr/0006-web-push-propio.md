# ADR-0006 · Web Push estándar con VAPID, sin proveedor externo

**Estado:** aceptada

## Contexto

Los datos de uso mostraban tareas olvidadas: la app solo servía si alguien la abría. Hacían falta
avisos en el móvil, en iOS y Android, sin coste y sin entregar datos de menores a terceros.

## Decisión

- **Web Push** (RFC 8030) con la librería `web-push` y claves **VAPID** generadas por el servidor
  y persistidas en `casa_meta` si no se configuran.
- Qué avisar se decide en funciones puras (`services/avisos.js`): eventos derivados de comparar
  el estado antes y después, nunca dirigidos a su autor y agrupados por destinatario.
- Solo se aceptan suscripciones de servicios de push conocidos (evita SSRF). Las caducadas
  (`404`/`410`) se eliminan solas.
- El envío ocurre después de responder al cliente.

## Alternativas

- **Firebase Cloud Messaging / OneSignal**: dependencias y cuentas externas, datos fuera.
- **Correo o mensajería**: requiere direcciones o números de teléfono de los menores.

## Consecuencias

- ✅ Sin coste ni terceros con acceso al contenido de la casa.
- ⚠️ En iOS solo funciona con la PWA instalada en la pantalla de inicio (iOS 16.4+).
