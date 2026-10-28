# Guía de contribución

Gracias por querer mejorar Casa. Al participar aceptas el [código de conducta](CODE_OF_CONDUCT.md).

- **¿Un error?** Abre un issue con la plantilla *Informe de error*, idealmente reproducible en `/demo`.
- **¿Una idea?** Plantilla *Propuesta*. Mira antes la [hoja de ruta](docs/ROADMAP.md).
- **¿Un fallo de seguridad?** En privado: [SECURITY.md](SECURITY.md).

## Entorno

```bash
npm install
cp .env.example .env
npm test
npm start
```

Para la batería de la API hace falta un PostgreSQL vacío; ver [docs/PRUEBAS.md](docs/PRUEBAS.md).

## Flujo de trabajo

1. Rama desde `main`: `feat/…`, `fix/…`, `docs/…`.
2. Cambios pequeños y con pruebas.
3. `npm test` en verde (y `npm run test:http` si tocas rutas).
4. Pull request con la plantilla. La CI debe pasar antes de fusionar; en cada instalación,
   `main` despliega automáticamente en Render.

## Commits

[Conventional Commits](https://www.conventionalcommits.org/), en español y en imperativo:

```
feat: calendario semanal de tareas adaptado a los horarios
fix: recordatorio solo dentro de la ventana de la tarde
docs: referencia de la API
deps: Express 5
```

`feat!:` o un pie `BREAKING CHANGE:` para cambios incompatibles. Si el commit se ha hecho con
ayuda de un agente de IA, se indica con un pie `Co-Authored-By:`.

## Estilo

- Identificadores y comentarios en español, como el resto del código.
- Los comentarios explican el **porqué**, no el qué.
- La lógica de negocio va en `services/` como funciones puras; las rutas solo orquestan.
- Sin dependencias nuevas salvo necesidad clara: hoy hay cuatro.

## Añadir un campo al estado

Es el cambio más delicado. Sigue los pasos en orden:

1. Valor inicial en `estadoVacio()` (`services/estadoService.js`).
2. Saneado en `sanitizeState()` (`services/stateGuard.js`). **Lo que no se sanea se pierde al
   guardar.**
3. Si es de los padres, añadirlo a `SOLO_PADRES` en `applyChildLimits()`.
4. Para estados existentes, normalizar **en el servidor** y persistirlo; nunca solo en el
   cliente ([ADR-0004](docs/adr/0004-normalizacion-en-el-servidor.md)).
5. Comparar con `huella()`, no con `JSON.stringify`.
6. Datos de ejemplo en `public/demo.js`.
7. Pruebas en `test/`.
8. Documentarlo en [docs/MODELO-DE-DATOS.md](docs/MODELO-DE-DATOS.md).

## Añadir una ruta

- Bajo `/api` queda protegida por el código de casa automáticamente.
- Si es de padres: middleware `soloPapas`.
- Si la demo la usa: respuesta simulada en `public/demo.js`.
- Caso en `test/http/api.js` y entrada en [docs/API.md](docs/API.md).

## Decisiones de arquitectura

Un cambio que afecte a la estructura (persistencia, sincronización, seguridad, dependencias) se
acompaña de un ADR en [docs/adr](docs/adr/).

## Datos personales

No incluyas nombres reales, direcciones de despliegue, códigos de casa ni copias del estado en el
repositorio, ni en issues o pull requests. Si guardas notas privadas, usa `privado/`, que git
ignora. Los datos de ejemplo usan nombres inventados (Mateo, Pablo y Sara).
