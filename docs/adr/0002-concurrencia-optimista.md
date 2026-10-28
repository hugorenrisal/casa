# ADR-0002 · Concurrencia optimista con versión y fusión en el cliente

**Estado:** aceptada

## Contexto

Varios dispositivos editan el mismo documento ([ADR-0001](0001-documento-unico-de-estado.md)):
un hijo marca una tarea en el móvil mientras los padres aprueban otra en el ordenador. Con
"gana el último", uno de los dos cambios desaparecía sin aviso.

## Decisión

- Cada lectura devuelve `_version`; cada escritura la envía.
- El servidor escribe con `UPDATE … WHERE version = $n` y responde **`409 version_desfasada`** si
  otro escribió antes.
- Ante un `409`, el cliente recarga y **reaplica los campos de primer nivel que él cambió** sobre
  el estado fresco.
- La comprobación de versión va antes que las reglas de perfil, para que un dispositivo desfasado
  reciba un error reintentable y no un `403`.
- Sincronización por sondeo cada 4 s.

## Alternativas

- **Bloqueo pesimista**: inviable con clientes móviles que se cierran en cualquier momento.
- **WebSockets / SSE**: mejor latencia, pero el plan gratuito del hosting suspende el servidor y
  corta las conexiones; el sondeo es suficiente a esta escala.
- **CRDT o registro de operaciones**: la solución correcta a largo plazo, desproporcionada hoy.

## Consecuencias

- ✅ Ningún cambio se pierde en silencio.
- ✅ Implementación pequeña y comprobada en `test/concurrencia.test.js`.
- ⚠️ Si dos dispositivos cambian **el mismo campo de primer nivel** a la vez, prevalece uno (con
  aviso). Evolución prevista: acciones granulares ([hoja de ruta](../ROADMAP.md#deuda-técnica)).
