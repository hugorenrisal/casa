# ADR-0009 · Copias de seguridad en tres capas

**Estado:** aceptada

## Contexto

Al principio las copias solo existían si alguien pulsaba un botón, y una sola petición de
reinicio borraba la casa sin vuelta atrás.

## Decisión

| Capa | Momento | Retención | Ubicación |
|---|---|---|---|
| Diaria | primera lectura del día, antes de aplicar ciclos | 30 | `casa_copias` |
| Preventiva | antes de `reset`, `restore` o restaurar copia | 10 | `casa_copias` |
| Externa | cada noche | una por día | rama `copias` del repositorio |

Un índice único parcial garantiza una sola copia diaria aunque lleguen peticiones simultáneas.

## Consecuencias

- ✅ Toda operación destructiva es reversible con otra operación.
- ✅ La pérdida total de la base de datos cuesta como máximo un día.
- ⚠️ La rama `copias` contiene datos reales de la casa: el repositorio debe ser privado, o la
  copia externa debe ir a otro destino privado.
