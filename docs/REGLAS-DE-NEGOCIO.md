# Reglas de negocio

Especificación funcional del producto. Cada regla está implementada en el servidor y cubierta
por pruebas; la columna **Dónde** apunta al código que la hace cumplir.

- [Perfiles](#1-perfiles)
- [Tareas](#2-tareas)
- [Paga](#3-paga)
- [Puntos, niveles y rachas](#4-puntos-niveles-y-rachas)
- [Premios](#5-premios)
- [Mercado entre hermanos](#6-mercado-entre-hermanos)
- [Calendario](#7-calendario)
- [Menús](#8-menús)
- [Avisos](#9-avisos)
- [Ciclos de tiempo](#10-ciclos-de-tiempo)
- [Matriz de permisos](#11-matriz-de-permisos)

---

## 1. Perfiles

Cuatro perfiles fijos definidos en `public/perfiles.js`: **tres hijos** (experiencia móvil) y
**el panel de los padres** (experiencia de escritorio). No hay registro ni contraseñas: cada
dispositivo elige perfil una vez y lo recuerda.

Elegir perfil no autentica. Es una decisión de producto: el objetivo es que un niño use la app
sin fricción, y las reglas que importan las hace cumplir el servidor
([ADR-0003](adr/0003-perfiles-fijos-sin-cuentas.md)).

## 2. Tareas

| Tipo | Ejemplo | Registro | Validación | Cuenta para |
|---|---|---|---|---|
| **Fija diaria** | Hacer la cama | una casilla por día | no | paga, puntos, racha |
| **Fija semanal** | Poner una lavadora | "Hecha" | sí | paga, puntos |
| **Adicional** | Limpiar el baño (40 pts) | "Hecha" | sí | dinero extra, puntos |

- Las fijas son las mismas para todos los hijos.
- Las adicionales se **reparten cada mes** de forma ponderada: cada tarea pesa sus puntos y cada
  hijo recibe carga en proporción a su disponibilidad (normal = 1, reducida = 0,6,
  mínima = 0,3). El reparto es determinista. — `public/index.html` (`generateMonth`),
  `test/dinero.test.js`
- Una tarea entregada queda pendiente de validación hasta que los padres la aprueban o la
  devuelven.

## 3. Paga

La app no mueve dinero: calcula lo que corresponde.

| Cumplimiento de las fijas en la semana | Resultado |
|---|---|
| < 75 % | sin paga fija |
| ≥ 75 % | paga fija (8 € por defecto) |
| 100 % | paga fija **+** adicionales aprobadas × valor del punto (0,05 € por defecto) |

El 100 % es la condición que **desbloquea** el dinero de las adicionales.

*Unidades:* cada día marcado de una diaria vale 1 y cada semanal aprobada vale 1. Con 3 diarias y
1 semanal hay 22 unidades semanales; el 75 % son 17. — `services/puntos.js`,
`test/dinero.test.js`

## 4. Puntos, niveles y rachas

- **Puntos del mes** = unidades de fijas × 2 + puntos de adicionales aprobadas ± primas del
  mercado. — `services/puntos.js`
- **Nivel** = uno cada 100 puntos.
- **Racha** = días consecutivos con todas las diarias marcadas. El día en curso no la rompe hasta
  que termina; se arrastra entre semanas si el domingo estaba completo. **La calcula siempre el
  servidor.** — `services/rachas.js`, `test/rachas.test.js`

## 5. Premios

1. Los padres definen premios con coste en puntos y stock opcional.
2. Un hijo pide uno: sus puntos quedan **reservados** al instante, de modo que no puede gastar
   dos veces los mismos.
3. Los padres lo conceden (se consumen los puntos y baja el stock) o lo deniegan (se liberan).

## 6. Mercado entre hermanos

Un hijo puede ofrecer una de sus adicionales:

- **por puntos** ("te doy 8 puntos si la haces tú"), o
- **por trueque** con otra tarea.

Otro hermano la solicita o propone el cambio y **el vendedor decide**. Las primas pactadas solo
se liquidan cuando la tarea acaba aprobada. El mercado se vacía al cerrar el mes. —
`test/mercado.test.js`

## 7. Calendario

Responde a la pregunta *¿cuándo hace cada uno sus tareas?*

- **Horarios**: bloques ocupados de cada hijo (colegio, actividades) con días y horas.
- **Franjas**: mañana 09–14, tarde 14–20, noche 20–23. Una franja está libre si quedan al menos
  **45 minutos** sin bloques.
- **Planificador** (`planificarSemana`), determinista y sin azar:
  1. solo coloca lo que falta y respeta lo colocado a mano;
  2. ordena adicionales por puntos (desc.) y después fijas semanales;
  3. elige la franja libre que minimiza, por este orden lexicográfico: tareas de ese hijo ese
     día, tareas totales en esa franja y día, caer en fin de semana y preferencia de franja
     (tarde primero);
  4. devuelve lo que no cabe.
- Una tarea colocada en franja ocupada se marca como **conflicto**, pero se permite (los padres
  deciden).
- La agenda es una **plantilla** que se repite cada semana y no se reinicia con los ciclos.
- Las diarias no entran en el calendario: son de todos los días.

— `test/calendario.test.js`

## 8. Menús

Base de platos por comida (desayuno, comida, cena) y una semana con un plato por comida; se puede
generar al azar desde la base. — `test/menus.test.js`

## 9. Avisos

Web Push en los dispositivos que lo activen. En iOS requiere la app instalada en la pantalla de
inicio (iOS 16.4+).

**Por eventos** (nunca a quien hizo el cambio; varios para la misma persona se agrupan en uno):

| Evento | Destinatario |
|---|---|
| Un hijo entrega una semanal o adicional | padres |
| Los padres aprueban / devuelven | ese hijo |
| Un hijo pide un premio | padres |
| Los padres lo conceden / deniegan | ese hijo |
| Un hermano solicita una tarea en el mercado | el vendedor |
| El vendedor acepta / rechaza | el solicitante |

**Recordatorio diario** (17:00–21:30, una vez al día):

- a cada hijo, lo planificado para hoy sin hacer y las diarias sin marcar; si no tiene nada
  pendiente, no recibe nada;
- a los padres, el número de elementos pendientes de validación.

— `services/avisos.js`, `test/avisos.test.js`

## 10. Ciclos de tiempo

Los decide el reloj del servidor en la zona `TZ_FAMILIA`.

| Ciclo | Efecto |
|---|---|
| Cada lunes | se vacían las casillas de las fijas; la racha se arrastra si el domingo estaba completo; las adicionales no se tocan |
| Cada día 1 | lo pendiente pasa a *vencida*; se archiva el resumen del mes (puntos, hechas, vencidas por hijo); se vacían reparto y mercado; calendario y horarios se conservan |

— `services/ciclos.js`, `test/ciclos.test.js`, `test/cierre-mes.test.js`

## 11. Matriz de permisos

| Acción | Hijo | Padres |
|---|:-:|:-:|
| Marcar sus diarias | ✅ | ✅ |
| Entregar sus semanales y adicionales | ✅ | ✅ |
| Aprobar, devolver, marcar vencida | ❌ | ✅ |
| Modificar tareas de otro hijo | ❌ | ✅ |
| Pedir premios, operar en el mercado | ✅ | — |
| Paga, valor del punto, catálogos, premios | ❌ | ✅ |
| Calendario y horarios | lectura | ✅ |
| Rachas e historial | lectura | lectura (los escribe el servidor) |
| Copias, reiniciar, restaurar, recordatorios | ❌ | ✅ |

Se aplica en `applyChildLimits()` y `soloPapas` independientemente de lo que haga el cliente.
— `services/stateGuard.js`, `test/permisos.test.js`
