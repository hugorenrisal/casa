<div align="center">

<img src="docs/img/hero.png" alt="Casa: panel de los padres en el ordenador y la app de un hijo en el móvil" width="100%" />

# 🏠 Casa

**Gestión doméstica para familias: tareas, paga semanal, calendario según los horarios de cada
hijo, menús y avisos en el móvil.**

Aplicación web progresiva (PWA) con backend Node.js y PostgreSQL, construida de principio a fin
con desarrollo asistido por IA y en uso diario en un hogar real.

[![Pruebas](https://github.com/hugorenrisal/casa/actions/workflows/pruebas.yml/badge.svg)](https://github.com/hugorenrisal/casa/actions/workflows/pruebas.yml)
[![Licencia MIT](https://img.shields.io/badge/licencia-MIT-blue)](LICENSE)
![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-5-000000?logo=express&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)
![PWA](https://img.shields.io/badge/PWA-Web%20Push-5A0FC8?logo=pwa&logoColor=white)
[![Hecho con Claude Code](https://img.shields.io/badge/hecho%20con-Claude%20Code-D97757)](docs/DESARROLLO-CON-IA.md)

[Desplegar](#-desplegar-en-render) · [Probar la demo](#-probar-la-demo) ·
[Documentación](#-documentación) · [Cómo se hizo con IA](docs/DESARROLLO-CON-IA.md)

</div>

> [!IMPORTANT]
> **Casa necesita un servidor: se despliega en [Render](https://render.com).** No es una página
> estática (no funciona en GitHub Pages): la API guarda el estado, aplica las reglas de cada
> perfil, decide los cambios de semana y de mes y envía los avisos. Render (servidor) y
> [Neon](https://neon.tech) (PostgreSQL) tienen plan gratuito, y el repositorio incluye un
> [`render.yaml`](render.yaml) para desplegar con un clic. Pasos en
> [Desplegar en Render](#-desplegar-en-render) y guía completa en
> [docs/DESPLIEGUE-EN-RENDER.md](docs/DESPLIEGUE-EN-RENDER.md).

---

## Índice

- [El problema](#-el-problema)
- [Capturas](#-capturas)
- [Funcionalidades](#-funcionalidades)
- [Probar la demo](#-probar-la-demo)
- [Desplegar en Render](#-desplegar-en-render)
- [Desarrollo local](#-desarrollo-local)
- [Arquitectura](#-arquitectura)
- [Pruebas](#-pruebas)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Documentación](#-documentación)
- [Hecho con IA](#-hecho-con-ia)
- [Hoja de ruta](#-hoja-de-ruta)
- [Contribuir](#-contribuir)
- [Licencia](#-licencia)

## 💡 El problema

En una casa con varios hijos, el reparto de tareas suele vivir en la memoria de los padres: quién
hizo qué, cuánto le toca de paga a cada uno, qué día puede cada uno según el colegio o las
actividades. Las apps genéricas de listas de tareas no modelan nada de eso.

**Casa** convierte esas reglas en un sistema: cada hijo ve en su móvil lo que le toca hoy, los
padres validan desde el ordenador, la paga se calcula sola y la app **avisa** cuando hay algo que
hacer o que aprobar.

## 📸 Capturas

Datos de ejemplo de la demo; los nombres son inventados.

<table>
  <tr>
    <td width="33%"><img src="docs/img/capturas/movil-inicio.png" alt="Inicio de un hijo: tareas de hoy, paga y progreso" /></td>
    <td width="33%"><img src="docs/img/capturas/movil-semana.png" alt="Semana de un hijo organizada según su horario" /></td>
    <td width="33%"><img src="docs/img/capturas/movil-logros.png" alt="Logros, nivel y rachas" /></td>
  </tr>
  <tr>
    <td align="center"><b>Hoy te toca</b><br/>Tareas del día por franja</td>
    <td align="center"><b>Mi semana</b><br/>Según colegio y actividades</td>
    <td align="center"><b>Logros</b><br/>Niveles, rachas e insignias</td>
  </tr>
</table>

<table>
  <tr>
    <td width="50%"><img src="docs/img/capturas/escritorio-calendario.png" alt="Calendario semanal con planificador y conflictos" /></td>
    <td width="50%"><img src="docs/img/capturas/escritorio-hijos.png" alt="Comparativa de los hijos en el panel de los padres" /></td>
  </tr>
  <tr>
    <td align="center"><b>Calendario</b><br/>Planificador por huecos libres con detección de conflictos</td>
    <td align="center"><b>Los hijos</b><br/>Progreso, paga y puntos de un vistazo</td>
  </tr>
</table>

Más capturas en [`docs/img/capturas`](docs/img/capturas/).

## ✨ Funcionalidades

| Área | Qué hace |
|---|---|
| **Tareas** | Tres tipos: fijas diarias, fijas semanales (con validación) y adicionales con puntos, repartidas cada mes de forma ponderada según la disponibilidad de cada hijo |
| **Economía** | Paga semanal por umbrales (75 % desbloquea la paga fija, 100 % desbloquea las adicionales); valor del punto configurable |
| **Gamificación** | Puntos, niveles, rachas calculadas en el servidor, insignias y premios canjeables con reserva de puntos |
| **Mercado interno** | Los hijos intercambian tareas adicionales por puntos o por trueque, con aceptación explícita |
| **Calendario** | Horarios de cada hijo (colegio, actividades), plantilla semanal por franjas y planificador determinista que coloca las tareas en huecos libres y detecta conflictos |
| **Menús** | Base de platos y menú semanal generable |
| **Avisos push** | Web Push estándar (VAPID) sin servicios de terceros: entregas, aprobaciones, premios, mercado y un recordatorio diario por la tarde |
| **Copias de seguridad** | Diaria en base de datos, automática antes de cada operación destructiva y, opcionalmente, nocturna en GitHub |
| **Multidispositivo** | Sincronización por sondeo con concurrencia optimista: ningún dispositivo pisa en silencio los cambios de otro |
| **Demo** | `/demo` ejecuta la app completa con datos ficticios y un servidor simulado en el navegador |
| **Accesibilidad** | Contrastes WCAG AA, foco visible, `prefers-reduced-motion` y etiquetas ARIA |

Dos experiencias sobre el mismo código: **móvil primero** para los hijos y **escritorio
primero** para el panel de los padres.

## 🎮 Probar la demo

La demo no necesita base de datos ni cuenta en ningún servicio:

```bash
git clone https://github.com/hugorenrisal/casa.git
cd casa
npm install
npm start
```

Abre <http://localhost:3000/demo> y cambia de perfil con el botón de la barra inferior. Nada de lo
que toques se guarda.

## 🚀 Desplegar en Render

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/hugorenrisal/casa)

Para tener **tu casa** funcionando necesitas tres cosas, todas con plan gratuito:

| Pieza | Servicio | Para qué |
|---|---|---|
| Código | **GitHub** | Tu copia del repositorio, con los nombres de tu familia |
| Base de datos | **[Neon](https://neon.tech)** | PostgreSQL donde vive el estado de la casa |
| Servidor | **[Render](https://render.com)** | Ejecuta la API y sirve la app con HTTPS |

**Paso a paso** (15–20 minutos):

1. **Crea tu copia.** *Use this template → Create a new repository*.
2. **Pon los nombres de tu casa** en [`public/perfiles.js`](public/perfiles.js) (`name`, `emoji`
   y `color` de cada hijo) y haz commit.
3. **Crea la base de datos** en [Neon](https://console.neon.tech): *New project* → región
   Frankfurt → *Connect* → copia la cadena de conexión con *pooling*.
4. **Elige un código de casa** con `npm run codigo` (o una frase de 12+ caracteres). Es la llave
   de la app: sin él, cualquiera con la dirección podría entrar.
5. **Crea el servicio en Render:** [dashboard.render.com](https://dashboard.render.com) → *New →
   Blueprint* → elige tu copia. Render lee [`render.yaml`](render.yaml) y solo te pide
   `DATABASE_URL` (paso 3) y `CODIGO_CASA` (paso 4) → *Apply*.
6. **Comprueba** `https://<tu-servicio>.onrender.com/api/health` →
   `{"ok":true,"db":true,"protegida":true}`.
7. **Instálala en los móviles:** panel de los papás → *Economía → Acceso → Copiar enlace*, ábrelo
   en cada móvil y *Añadir a pantalla de inicio*.

Opcional: el recordatorio diario y la copia nocturna se activan con la variable `CASA_URL` en
GitHub Actions. Todo el detalle, las variables de entorno y los problemas frecuentes están en
**[docs/DESPLIEGUE-EN-RENDER.md](docs/DESPLIEGUE-EN-RENDER.md)**.

> [!NOTE]
> El botón **Deploy to Render** de arriba despliega *este* repositorio tal cual (útil para
> probar). Para tu casa usa tu copia (pasos 1–5): así recibes tus cambios en cada push.
> En el plan gratuito el servidor se duerme tras 15 minutos sin uso y la primera carga tarda
> alrededor de un minuto.

## 🛠 Desarrollo local

**Requisitos:** Node.js 18 o superior (CI usa 22) y un PostgreSQL accesible (local, Docker o Neon).

```bash
npm install
cp .env.example .env      # rellena DATABASE_URL
npm start                 # http://localhost:3000
```

Las migraciones se aplican solas al arrancar. PostgreSQL desechable con Docker:

```bash
docker run -d --name casa-pg -e POSTGRES_PASSWORD=prueba -e POSTGRES_DB=casa -p 55432:5432 postgres:16-alpine
```

```dotenv
DATABASE_URL=postgres://postgres:prueba@localhost:55432/casa?sslmode=disable
```

| Comando | Qué hace |
|---|---|
| `npm start` | Arranca el servidor |
| `npm test` | 147 pruebas de lógica, sin base de datos ni red |
| `npm run test:http` | 45 comprobaciones contra un servidor real y un PostgreSQL vacío |
| `npm run codigo` | Genera un código de casa aleatorio |

Variables de entorno: [`.env.example`](.env.example) y
[docs/DESPLIEGUE-EN-RENDER.md](docs/DESPLIEGUE-EN-RENDER.md#variables-de-entorno).

## 🧱 Arquitectura

```mermaid
flowchart LR
  subgraph Dispositivos
    A["PWA<br/>public/index.html"]
    SW["Service worker<br/>public/sw.js"]
  end
  subgraph Servidor["Servidor (Render)"]
    E["Express 5<br/>routes/ + services/"]
  end
  DB[("PostgreSQL (Neon)<br/>documento de estado<br/>+ copias, avisos, meta")]
  GH["GitHub Actions<br/>CI · recordatorio · copia"]
  PUSH["Servicios Web Push<br/>Apple · Google · Mozilla"]

  A -- "REST /api (sondeo 4 s)" --> E
  E --> DB
  GH -- "programado" --> E
  E -- "VAPID" --> PUSH --> SW
```

Decisiones principales (detalle en [docs/adr](docs/adr/)):

- **Un único documento JSON** por casa en una fila de PostgreSQL, con **versión optimista**
  (`409 Conflict` ante una escritura desfasada y fusión por campo en el cliente).
- **El servidor es la autoridad**: sanea cada escritura, aplica las reglas de cada perfil, calcula
  rachas y decide los ciclos de semana y mes con su propio reloj.
- **Lógica de negocio pura** en `services/`, sin E/S, lo que permite probarla sin base de datos.
- **Dependencias mínimas**: cuatro en producción (`express`, `pg`, `dotenv`, `web-push`) y
  ninguna de desarrollo. Cliente sin framework.

| Capa | Tecnología |
|---|---|
| Servidor | Node.js 18+, Express 5 |
| Base de datos | PostgreSQL 16, migraciones SQL idempotentes |
| Cliente | HTML, CSS y JavaScript sin framework; PWA instalable |
| Avisos | Web Push (RFC 8030) con claves VAPID generadas y persistidas por el servidor |
| Pruebas | `node:test` (runner nativo) y batería HTTP extremo a extremo |
| CI/CD | GitHub Actions, Dependabot, Blueprint de Render con despliegue continuo |
| Android | Trusted Web Activity (Bubblewrap) preparada para Google Play |

Más en [docs/ARQUITECTURA.md](docs/ARQUITECTURA.md).

## ✅ Pruebas

- **Lógica** (`test/*.test.js`): dinero, reparto ponderado, saneado, permisos por perfil,
  rachas, cierre de mes, mercado, concurrencia, calendario, avisos, copias y código de acceso.
- **API** (`test/http/api.js`): rutas, códigos de estado, permisos, límite de intentos, copias y
  avisos, contra PostgreSQL real. Se niega a ejecutarse contra producción.
- **CI**: ambas suites corren en cada push a `main` y en cada pull request, junto con
  `npm audit --audit-level=high`.

Estrategia completa en [docs/PRUEBAS.md](docs/PRUEBAS.md).

## 📁 Estructura del repositorio

```
.
├── server.js              # Arranque, cabeceras de seguridad, montaje de rutas
├── render.yaml            # Blueprint de Render (despliegue en un clic)
├── routes/                # Capa HTTP: estado, avisos, copias
├── services/              # Lógica de negocio (pura) y acceso a datos
├── db/                    # Pool de PostgreSQL y migraciones SQL
├── public/                # PWA: app, perfiles, service worker, demo
├── test/                  # Pruebas de lógica y batería HTTP
├── casa-android/          # Empaquetado Android (TWA)
├── herramientas/          # Generación de iconos
├── docs/                  # Documentación técnica, ADR e imágenes
└── .github/               # CI, tareas programadas, Dependabot y plantillas
```

## 📚 Documentación

| Documento | Contenido |
|---|---|
| [Desplegar en Render](docs/DESPLIEGUE-EN-RENDER.md) | **Instalación paso a paso**: GitHub, Neon, Render, móviles y problemas frecuentes |
| [Manual de uso](docs/MANUAL-DE-USO.md) | Guía para la familia: instalación en el móvil, avisos, uso mensual |
| [Arquitectura](docs/ARQUITECTURA.md) | Componentes, ciclo de vida de una petición, sincronización, ciclos de tiempo |
| [Reglas de negocio](docs/REGLAS-DE-NEGOCIO.md) | Especificación funcional: tareas, dinero, puntos, mercado, calendario, avisos, permisos |
| [Referencia de la API](docs/API.md) | Endpoints, cabeceras, respuestas y códigos de error |
| [Modelo de datos](docs/MODELO-DE-DATOS.md) | Documento de estado, tablas y migraciones |
| [Pruebas](docs/PRUEBAS.md) | Estrategia, suites y CI |
| [Despliegue y operación](docs/DESPLIEGUE.md) | Variables, tareas programadas, copias y recuperación |
| [Decisiones de arquitectura](docs/adr/) | Diez ADR con el contexto y las alternativas de cada decisión |
| [Desarrollo con IA](docs/DESARROLLO-CON-IA.md) | Cómo se construyó el proyecto con un agente de IA |
| [Hoja de ruta](docs/ROADMAP.md) | Estado por módulo y próximos pasos |

## 🤖 Hecho con IA

El proyecto se ha desarrollado con **[Claude Code](https://claude.com/claude-code)** (Anthropic)
como agente de programación, bajo dirección humana: el autor define el producto, toma las
decisiones y valida cada entrega en el uso real; el agente diseña, implementa, prueba y verifica
en el navegador y en el servidor.

El resultado es un código con pruebas, decisiones documentadas y commits convencionales con
coautoría explícita. El proceso, los criterios de calidad y las lecciones aprendidas están en
[docs/DESARROLLO-CON-IA.md](docs/DESARROLLO-CON-IA.md).

## 🗺 Hoja de ruta

Versión **3.3.0**, estable. Próximos pasos: hucha con objetivo, exportación del calendario a
iCal, división del cliente en módulos y pruebas automáticas de interfaz. Detalle en
[docs/ROADMAP.md](docs/ROADMAP.md) y cambios en [CHANGELOG.md](CHANGELOG.md).

## 🤝 Contribuir

Las propuestas y los informes de error son bienvenidos: lee [CONTRIBUTING.md](CONTRIBUTING.md) y
el [código de conducta](CODE_OF_CONDUCT.md). Los fallos de seguridad se notifican en privado
según [SECURITY.md](SECURITY.md).

## 📄 Licencia

[MIT](LICENSE).
