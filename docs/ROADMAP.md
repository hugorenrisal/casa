# Hoja de ruta

Estado a 28 de octubre de 2026, versión 3.3.0. El porcentaje mide **"listo para el uso diario"**,
no cantidad de código.

## Estado por módulo

| Módulo | Estado | Notas |
|---|:-:|---|
| Motor de tareas y paga | 95 % | Umbrales, reparto ponderado, niveles, primas |
| Perfiles y permisos | 95 % | Reglas aplicadas en el servidor |
| Pruebas y CI | 95 % | 147 de lógica + 45 de API; falta automatizar la interfaz |
| Panel de padres | 90 % | Vista conjunta, ficha por hijo, calendario, copias, avisos |
| Rachas | 90 % | Calculadas en el servidor, con arrastre semanal |
| Mercado entre hermanos | 90 % | Aceptación explícita del vendedor |
| Copias de seguridad | 90 % | Tres capas, restauración con un clic |
| Código de casa | 90 % | Implementado; desactivado por decisión en uso privado |
| PWA e instalación | 90 % | Android e iOS, con avisos |
| Demo | 90 % | Sin ninguna petición a la API |
| Calendario | 85 % | Falta arrastrar y soltar |
| Avisos push | 85 % | Eventos y recordatorio diario |
| Premios | 85 % | Reserva de puntos, stock |
| Menús | 85 % | Falta lista de la compra |
| Concurrencia | 85 % | Fusión por campo de primer nivel |
| Accesibilidad | 75 % | WCAG AA en contraste; falta modo oscuro y revisión con lector de pantalla |
| Validación de tareas | 60 % | Falta prueba con foto |
| Logros | 55 % | Insignias fijas, sin progresión |

## Próximas funcionalidades

Ordenadas por impacto esperado en el uso real:

1. **Hucha con objetivo** — ahorro visible hacia una meta concreta.
2. **Exportación iCal por hijo** — las tareas en el calendario nativo del móvil, con sus avisos.
3. **Hora del recordatorio configurable** desde la app.
4. **Prueba con foto** al entregar una tarea.
5. **Lista de la compra** derivada del menú semanal.
6. **Calendario**: arrastrar y soltar; solicitudes de cambio de los hijos con aprobación.
7. **Resumen semanal** para los padres.
8. **Logros con progresión** y aviso al desbloquear.
9. **Modo oscuro.**

## Deuda técnica

| Elemento | Prioridad | Propuesta |
|---|:-:|---|
| Cliente en un único `index.html` (~2.700 líneas) | alta | Módulos ES con un paso de *bundle* mínimo |
| Sin pruebas automáticas de interfaz | alta | Playwright sobre `/demo` en CI |
| Fusión de conflictos por campo de primer nivel | media | Registro de acciones granulares en lugar del documento completo |
| `normalizarMenus()` (servidor) y `ensureShape()` (cliente) duplicados | media | Normalización solo en el servidor, como el calendario |
| Recordatorio dependiente de la puntualidad de GitHub | media | Planificador externo o tareas programadas del proveedor |
| Tablas de la versión con cuentas sin uso | baja | Migración de limpieza tras una copia |

## Seguridad (si se abre a más usuarios)

- Activar el código de casa.
- Política de privacidad con contacto real.
- Revisión previa a la publicación en Google Play.

Ver [SECURITY.md](../SECURITY.md).
