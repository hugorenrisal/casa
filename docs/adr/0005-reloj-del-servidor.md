# ADR-0005 · El reloj del servidor decide los ciclos

**Estado:** aceptada

## Contexto

El cambio de semana vacía casillas y el cambio de mes archiva el historial y marca vencidas. Al
principio lo decidía el cliente con su fecha local: un móvil con la fecha mal configurada podía
reiniciar el mes de toda la casa.

## Decisión

- Las claves de día, semana ISO y mes se calculan en el servidor con la zona `TZ_FAMILIA`
  (`services/tiempo.js`).
- Los ciclos se aplican **de forma perezosa** en la primera lectura posterior
  (`leerEstadoAlDia`), lo que tolera que el servidor esté suspendido.
- Una escritura con un ciclo anterior se rechaza con `409 ciclo_desfasado`.

## Consecuencias

- ✅ Ningún dispositivo puede adelantar ni atrasar un ciclo.
- ✅ No hace falta un planificador para los ciclos.
- ⚠️ La zona horaria es por despliegue, no por usuario (coherente con una sola casa).
