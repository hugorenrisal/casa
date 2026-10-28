# ADR-0004 · Normalizar el estado solo en el servidor

**Estado:** aceptada

## Contexto

Al añadir campos (menús, calendario), los estados antiguos no los tienen. Si el **cliente**
completaba la forma en memoria sin guardarla, cada sondeo veía una diferencia con el servidor,
adoptaba lo remoto, lo volvía a completar y repintaba: un **bucle de repintado** cada 4 s.

Además, PostgreSQL (`jsonb`) reordena las claves de los objetos, así que comparar con
`JSON.stringify` daba "distinto" para objetos iguales y provocaba una escritura en cada lectura.

## Decisión

- Toda normalización se hace en el servidor (`ensureFixedShape`, `normalizarMenus`,
  `normalizarCalendario`) **y se persiste** en la lectura si hubo cambios.
- El cliente nunca escribe al leer: usa valores por defecto (`S.agenda || []`).
- Las comparaciones usan `huella()`, independiente del orden de las claves.

## Consecuencias

- ✅ Estados antiguos y nuevos convergen sin intervención.
- ✅ El sondeo es silencioso en reposo: misma versión en lecturas consecutivas.
- ⚠️ Queda una duplicación histórica: `normalizarMenus()` (servidor) y `ensureShape()` (cliente)
  deben coincidir. Pendiente de eliminar.
