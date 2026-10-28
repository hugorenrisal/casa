# ADR-0001 · Un único documento JSON de estado por casa

**Estado:** aceptada

## Contexto

Una instalación sirve a una sola casa: cuatro perfiles, unas decenas de tareas y unos pocos KB de
datos. Casi todas las pantallas necesitan casi todo el estado (el panel de padres muestra a la
vez tareas, paga, calendario y mercado), y las reglas de negocio cruzan entidades constantemente.

## Decisión

Guardar el estado completo como **un documento JSONB** en una única fila de `casa_state`
(`CHECK (id = 1)`). El cliente lo descarga entero, lo modifica y lo envía entero. Las tablas
aparte se reservan para lo que no es de la casa: suscripciones push (de cada dispositivo), copias
(fotografías del documento) y metadatos del servidor.

## Alternativas

- **Modelo relacional normalizado** (tareas, asignaciones, casillas…): más consultas, más
  migraciones y más código de mapeo para un volumen que nunca lo necesitará.
- **Almacén clave-valor por entidad**: complica las operaciones atómicas que tocan varias
  entidades (cierre de mes, mercado).

## Consecuencias

- ✅ Copiar, restaurar y descargar la casa es copiar un documento.
- ✅ Las reglas de negocio son funciones `estado → estado`, fáciles de probar.
- ✅ Una sola escritura atómica por cambio.
- ⚠️ La concurrencia hay que resolverla a nivel de documento ([ADR-0002](0002-concurrencia-optimista.md)).
- ⚠️ Cada campo nuevo exige saneado explícito y normalización en el servidor
  ([ADR-0004](0004-normalizacion-en-el-servidor.md)).
- ⚠️ No escala a muchas casas por despliegue; no es un objetivo.
