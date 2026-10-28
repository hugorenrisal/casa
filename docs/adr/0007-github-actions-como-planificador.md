# ADR-0007 · GitHub Actions como planificador de tareas

**Estado:** aceptada

## Contexto

El recordatorio de la tarde y la copia nocturna necesitan ejecutarse a una hora. El plan
gratuito del hosting suspende el servidor sin tráfico y no ofrece tareas programadas.

## Decisión

- Workflows programados de GitHub Actions llaman a la API (y de paso despiertan el servidor).
- Como GitHub puede retrasar las ejecuciones horas, el recordatorio se intenta **cinco veces** por
  la tarde y el servidor solo actúa dentro de su **ventana horaria** (17:00–21:30) y **una vez al
  día**, registrado de forma atómica en `casa_meta`.
- La copia nocturna guarda el estado en la rama `copias`, fuera del proveedor de base de datos.

## Alternativas

- **Plan de pago del hosting** con tareas programadas: coste recurrente.
- **Planificador externo** (p. ej. cron-job.org): otra cuenta y otro secreto que gestionar. Queda
  como alternativa si los retrasos son frecuentes.

## Consecuencias

- ✅ Coste cero y todo versionado en el repositorio.
- ✅ Idempotente: repetir la llamada no duplica avisos.
- ⚠️ Si todos los intentos llegan tarde, ese día no hay recordatorio (se prefiere a avisar de
  noche).
