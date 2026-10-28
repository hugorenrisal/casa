# Registro de cambios

Formato basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/); el proyecto sigue
[Versionado Semántico](https://semver.org/lang/es/). Las versiones anteriores a 3.3.0 se
desarrollaron en un repositorio privado y se resumen aquí a partir de su historial.

## [Sin publicar]

## [3.3.0] — 2026-10-28 · Primera versión pública

### Añadido
- Despliegue en un clic con [`render.yaml`](render.yaml) (Blueprint de Render) y guía paso a
  paso en [docs/DESPLIEGUE-EN-RENDER.md](docs/DESPLIEGUE-EN-RENDER.md).
- Documentación técnica completa: arquitectura, API, modelo de datos, reglas de negocio, pruebas,
  despliegue, manual de uso, hoja de ruta, desarrollo con IA y diez ADR.
- Licencia MIT, código de conducta, guía de contribución, política de seguridad, plantillas de
  issues y pull requests, `.editorconfig`, `.gitattributes` y `.nvmrc`.
- Capturas de la demo en el README.

### Cambiado
- Perfiles de ejemplo con nombres inventados (Mateo, Pablo y Sara); los títulos que nombraban
  a los hijos salen ahora de `public/perfiles.js`.
- Las tareas programadas solo se ejecutan si existe la variable `CASA_URL`; la copia nocturna
  se niega a ejecutarse en repositorios públicos.
- La configuración de Android usa un marcador (`TU-SERVICIO`) en lugar de una dirección fija.

## 3.2.0 — 2026-10-28

### Añadido
- Batería de pruebas de la API contra PostgreSQL real en CI (`npm run test:http`).

### Cambiado
- Express 5, dotenv 18, pg 8.23 y acciones de GitHub v7.
- El recordatorio de la tarde solo se envía entre 17:00 y 21:30 y se intenta cinco veces.
- Paleta con contraste WCAG AA, tamaño mínimo de letra de 11 px, foco visible,
  `prefers-reduced-motion` y etiquetas ARIA en botones de icono.
- El código de casa pasa a ser opcional y sin avisos de alerta.

### Corregido
- `/demo` devolvía error con Express 5.
- Los errores del cliente (JSON mal formado, cuerpo excesivo) devolvían `500` en lugar de `4xx`.

## 3.1.0 — 2026-10-26

### Añadido
- Calendario semanal: horarios de cada hijo, plantilla por franjas, planificador determinista y
  detección de conflictos.
- Avisos push (Web Push + VAPID) por eventos y recordatorio diario.
- Código de casa con límite de intentos.
- Copias de seguridad diarias, preventivas y nocturnas fuera del servidor.
- Dependabot y `npm audit` en CI.

### Corregido
- Cualquier petición sin cabeceras podía reiniciar el estado.
- La copia descargada no pasaba por el control de acceso.
- Restaurar dejaba el cliente sin versión.

## 3.0.0 — 2026-09-03

### Cambiado
- **Incompatible:** las cuentas, familias e invitaciones se sustituyen por cuatro perfiles fijos
  ([ADR-0003](docs/adr/0003-perfiles-fijos-sin-cuentas.md)). Migración automática de los datos.

### Eliminado
- Registro, contraseñas, verificación y envío de correo.

## 2.0.0 — 2026-08-15

### Añadido
- App instalable (PWA) y preparación para Google Play (TWA).
- Demo pública en `/demo` sin acceso a datos reales.
- Saneado del estado y permisos aplicados en el servidor.
- Rachas reales, canje de premios y confirmación en el mercado.
- Vencimientos automáticos e historial mensual.
- Concurrencia optimista entre dispositivos.
- Primeras pruebas automáticas (dinero, reparto, canje).

### Corregido
- Bucle de repintado, ciclos decididos por el reloj del móvil y XSS al pintar nombres.
- Menús rotos en casas nuevas (`[object Object]`).

## 1.0.0 — 2026-06-04

### Añadido
- Primera versión: tareas, paga, premios y menús con cuentas, familias e invitaciones por correo.

[Sin publicar]: https://github.com/hugorenrisal/casa/compare/v3.3.0...HEAD
[3.3.0]: https://github.com/hugorenrisal/casa/releases/tag/v3.3.0
