# Estrategia de pruebas

| Nivel | Herramienta | Cantidad | Qué cubre | Cuándo corre |
|---|---|---:|---|---|
| Lógica de dominio | `node:test` | 147 | Reglas de negocio puras, sin base de datos ni red | `npm test`, CI |
| API extremo a extremo | `test/http/api.js` | 45 | Capa HTTP completa contra PostgreSQL real | `npm run test:http`, CI |
| Interfaz | Navegador (manual asistido) | — | Pantallas en `/demo` a 375 px y escritorio, consola sin errores | antes de cada entrega |
| Producción | Comprobaciones de solo lectura | — | Versión estable entre lecturas, salud, sin escrituras por lectura | tras cada despliegue |

Sin dependencias de pruebas: se usa el runner nativo de Node.

## Pruebas de lógica

```bash
npm test
```

| Suite | Pruebas | Comportamiento verificado |
|---|---:|---|
| `dinero.test.js` | 20 | Umbrales del 75 % / 100 %, valor del punto, reparto ponderado por disponibilidad, canje |
| `avisos.test.js` | 18 | Quién recibe cada aviso, nunca el autor, agrupación, recordatorio del día |
| `calendario.test.js` | 18 | Saneado de horarios y agenda, minutos libres, planificador determinista, conflictos, permisos |
| `permisos.test.js` | 16 | Límites del perfil de hijo: qué puede y qué no puede cambiar |
| `rachas.test.js` | 14 | Días completos, arrastre entre semanas, racha calculada solo en el servidor |
| `perfiles.test.js` | 12 | Perfiles fijos y traducción de ids de la versión con cuentas |
| `mercado.test.js` | 10 | Ofertas, trueques, aceptación del vendedor, liquidación de primas |
| `codigo-casa.test.js` | 8 | Código de acceso, normalización, límite de intentos con reloj simulado |
| `menus.test.js` | 8 | Normalización de platos y menús (incluido el caso `[object Object]`) |
| `cierre-mes.test.js` | 7 | Vencidas, archivo del historial, vaciado del mes |
| `concurrencia.test.js` | 6 | Versión optimista y fusión por campo del cliente |
| `ciclos.test.js` | 5 | Cambio de semana y mes con el reloj del servidor |
| `copias.test.js` | 5 | Copia diaria única, retención, restauración |

### Probar el código del cliente

La lógica del cliente vive en `public/index.html`. `test/_cliente.js` extrae sus funciones y las
evalúa en un contexto aislado, de modo que el planificador del calendario, la fusión de
conflictos o el reparto mensual se prueban igual que el código del servidor, sin navegador.

## Pruebas de la API

```bash
docker run -d --name casa-pg -e POSTGRES_PASSWORD=prueba -e POSTGRES_DB=casa -p 55432:5432 postgres:16-alpine
DATABASE_URL=postgres://postgres:prueba@localhost:55432/casa?sslmode=disable CODIGO_CASA=PRUE-BA12-3456 npm start
npm run test:http   # en otra terminal
```

Cubre: salud pública, rechazo sin código (`401`), límite de intentos (`429`), lectura y
escritura con versión (`409`), límites de hijo (`403`), rutas solo de padres, JSON mal formado
(`400`), cuerpo excesivo (`413`), reset y restore con copia previa, restauración de copias,
recordatorio (`simular`, ventana horaria, una vez al día), suscripciones push y `/demo`.

Escribe y reinicia el estado, así que **se niega a ejecutarse contra el despliegue de
producción**.

> Esta batería detectó que la actualización a Express 5 rompía `/demo`, algo que las pruebas de
> lógica no podían ver. Desde entonces corre en CI en cada cambio.

## Integración continua

`.github/workflows/pruebas.yml`, en cada `push` a `main` y en cada pull request (incluidas las de
Dependabot):

| Trabajo | Pasos |
|---|---|
| `logica` | `npm ci` → `npm test` → `npm audit --audit-level=high` |
| `api` | servicio `postgres:16-alpine` → arranque del servidor → espera a `/api/health` → `npm run test:http` |

`npm audit` hace fallar la CI ante una vulnerabilidad alta o crítica aunque nadie haya tocado el
código.

## Verificación en el navegador

Antes de cada entrega con cambios visibles:

1. `/demo` en local (no necesita base de datos).
2. Recorrido de todas las pantallas de padres e hijo a 375 px y en escritorio.
3. Ningún `undefined`, `NaN` ni `[object Object]` en pantalla; consola sin errores.
4. Cero peticiones a `/api/` en modo demo.

## Verificación en producción

Tras desplegar, solo lectura:

- `GET /api/health` responde `ok`.
- Tres `GET /api/state` seguidos devuelven la **misma versión** (no hay escrituras por lectura).
- Los campos nuevos aparecen con su forma por defecto.

## Añadir pruebas

- Una regla nueva del dominio lleva su prueba en la suite correspondiente.
- Un error corregido lleva una prueba que lo reproduce.
- Una ruta nueva se añade a `test/http/api.js`.
