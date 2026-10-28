# Desplegar Casa en Render, paso a paso

> [!IMPORTANT]
> **Casa necesita un servidor.** No es una página estática: la API guarda el estado de la casa,
> aplica las reglas de cada perfil, decide los cambios de semana y de mes con su propio reloj y
> envía los avisos push. Por eso no funciona en GitHub Pages ni abriendo `index.html` en el
> navegador. Esta guía la monta en **[Render](https://render.com)** (servidor web) con la base
> de datos en **[Neon](https://neon.tech)** (PostgreSQL). Los dos tienen plan gratuito.

| | |
|---|---|
| Tiempo | 15–20 minutos |
| Coste | 0 €/mes |
| Necesitas | Cuenta de GitHub, de Render y de Neon (las dos últimas se crean con la de GitHub) |
| Resultado | `https://<tu-servicio>.onrender.com` con HTTPS, instalable en los móviles |

```mermaid
flowchart LR
  A[1 · Tu copia<br/>en GitHub] --> B[2 · Base de datos<br/>en Neon]
  B --> C[3 · Código<br/>de casa]
  C --> D[4 · Servicio<br/>en Render]
  D --> E[5 · Comprobar]
  E --> F[6 · Instalar en<br/>los móviles]
  F -.-> G[7 · Tareas programadas<br/>opcional]
```

- [Antes de empezar: pruébala en local](#antes-de-empezar-pruébala-en-local)
- [1. Crea tu copia del repositorio](#1-crea-tu-copia-del-repositorio)
- [2. Crea la base de datos en Neon](#2-crea-la-base-de-datos-en-neon)
- [3. Elige el código de casa](#3-elige-el-código-de-casa)
- [4. Crea el servicio en Render](#4-crea-el-servicio-en-render)
- [5. Comprueba que funciona](#5-comprueba-que-funciona)
- [6. Instálala en los móviles](#6-instálala-en-los-móviles)
- [7. Opcional: recordatorio diario y copia nocturna](#7-opcional-recordatorio-diario-y-copia-nocturna)
- [Actualizar](#actualizar)
- [Límites del plan gratuito](#límites-del-plan-gratuito)
- [Problemas frecuentes](#problemas-frecuentes)

---

## Antes de empezar: pruébala en local

No hace falta base de datos para ver la app entera con datos de ejemplo:

```bash
git clone https://github.com/hugorenrisal/casa.git
cd casa
npm install
npm start
```

Abre `http://localhost:3000/demo`. Es la app completa con un servidor simulado en el navegador:
nada se guarda y no se hace ninguna petición a la API.

## 1. Crea tu copia del repositorio

1. En la página del repositorio, pulsa **Use this template → Create a new repository**.
2. Ponle nombre (por ejemplo `casa`) y elige la visibilidad:
   - **Privado** si vas a usar la copia nocturna del [paso 7](#7-opcional-recordatorio-diario-y-copia-nocturna)
     (guarda los datos de tu casa en el propio repositorio).
   - **Público** si solo quieres el servidor. Los datos viven en la base de datos, no en GitHub.
3. **Personaliza los perfiles** antes del primer uso. En tu copia, abre
   [`public/perfiles.js`](../public/perfiles.js) → icono del lápiz → cambia `name`, `emoji` y
   `color` de cada hijo → **Commit changes**.

```js
const HIJOS = [
  { id: "mateo", name: "Mateo", role: "child", color: "#2f9fd0", emoji: "🦊" },
  { id: "pablo", name: "Pablo", role: "child", color: "#2fae73", emoji: "🐢" },
  { id: "sara",  name: "Sara",  role: "child", color: "#e0588f", emoji: "🦋" },
];
```

> [!NOTE]
> Los `id` son internos y no se ven en la pantalla: puedes dejarlos como están. Si los cambias,
> hazlo **antes** de empezar a usar la app (el historial se guarda con ellos) y ajusta
> `test/perfiles.test.js`.

## 2. Crea la base de datos en Neon

Render también ofrece PostgreSQL gratis, pero **caduca a los 30 días**. Neon no caduca, y el
estado de una casa ocupa unos pocos kilobytes.

1. Entra en [console.neon.tech](https://console.neon.tech) con tu cuenta de GitHub.
2. **New project** → nombre `casa` → región **AWS Europe Central 1 (Frankfurt)**, la misma que
   el servicio de Render del paso 4, para que vayan rápido.
3. En el panel del proyecto pulsa **Connect**, deja activado **Connection pooling** y copia la
   cadena de conexión. Tiene esta forma:

   ```text
   postgresql://usuario:contraseña@ep-xxxx-pooler.eu-central-1.aws.neon.tech/neondb?sslmode=require
   ```

Guárdala: es la variable `DATABASE_URL`. No hace falta crear tablas: el servidor aplica las
migraciones solo al arrancar.

## 3. Elige el código de casa

Una vez desplegada, la app está en Internet. El **código de casa** (`CODIGO_CASA`) es la llave:
sin él, la API no responde. Cada móvil lo recibe una sola vez y lo recuerda.

- Genera uno aleatorio en tu ordenador: `npm run codigo` → p. ej. `K7QM-2XRT-9WPB`.
- O usa cualquier frase de 12 caracteres o más.

> [!WARNING]
> Si dejas `CODIGO_CASA` vacío, **cualquiera que conozca la dirección puede ver y cambiar los
> datos de tu casa**. Déjalo vacío solo para probar.

## 4. Crea el servicio en Render

El repositorio incluye [`render.yaml`](../render.yaml), un *Blueprint* que describe el servicio:
Node, plan gratuito, región Frankfurt, `npm ci`, `npm start`, comprobación de salud en
`/api/health` y despliegue en cada push. Así no tienes que configurar nada a mano.

### Opción A — Blueprint (recomendada)

1. Entra en [dashboard.render.com](https://dashboard.render.com) con tu cuenta de GitHub.
2. **New → Blueprint**.
3. **Connect a repository** → elige **tu copia** del paso 1. Si no aparece, pulsa
   *Configure GitHub* y dale acceso a ese repositorio.
4. Render lee `render.yaml` y muestra el servicio `casa`. Rellena las dos variables que pide:

   | Variable | Valor |
   |---|---|
   | `DATABASE_URL` | La cadena de Neon del paso 2 |
   | `CODIGO_CASA` | El código del paso 3 |

5. **Apply** (o **Deploy Blueprint**). La primera construcción tarda 2–3 minutos.

> [!TIP]
> El botón **Deploy to Render** del README hace lo mismo pero despliega **este** repositorio,
> no tu copia: sirve para probarla rápido, pero no recibirá tus cambios en `perfiles.js`.

### Opción B — Web Service manual

Si prefieres no usar el Blueprint: **New → Web Service** → tu repositorio, y:

| Campo | Valor |
|---|---|
| Name | `casa` (forma parte de la dirección) |
| Region | Frankfurt (EU Central) |
| Branch | `main` |
| Runtime | Node |
| Build Command | `npm ci` |
| Start Command | `npm start` |
| Instance Type | Free |
| Environment Variables | `DATABASE_URL`, `CODIGO_CASA`, `NODE_ENV=production`, `TZ_FAMILIA=Europe/Madrid` |
| Health Check Path | `/api/health` (en *Advanced*) |

La versión de Node la fija [`.nvmrc`](../.nvmrc) (22). Render asigna el puerto con la variable
`PORT`: no hay que configurarlo.

### Variables de entorno

| Variable | Obligatoria | Qué es |
|---|:-:|---|
| `DATABASE_URL` | sí | Cadena de conexión de PostgreSQL |
| `CODIGO_CASA` | muy recomendada | Código de acceso de la casa |
| `TZ_FAMILIA` | no | Huso horario que decide el cambio de día, semana y mes (`Europe/Madrid`) |
| `NODE_ENV` | no | `production` |
| `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY` | no | Claves de los avisos; si faltan, se generan y se guardan en la base de datos |
| `VAPID_SUBJECT` | no | Contacto ante Apple y Google; por defecto, la dirección del servicio |
| `ANDROID_FINGERPRINT`, `ANDROID_PACKAGE` | no | Solo si publicas la app en Google Play |

## 5. Comprueba que funciona

1. En Render → tu servicio → **Logs**, espera a ver `Casa escuchando en :10000`.
2. Abre `https://<tu-servicio>.onrender.com/api/health`. Debe responder:

   ```json
   { "ok": true, "db": true, "protegida": true }
   ```

   - `db: false` → `DATABASE_URL` falta o está mal copiada.
   - `protegida: false` → no hay `CODIGO_CASA`.
3. Abre `https://<tu-servicio>.onrender.com`, escribe el código y elige el perfil **Papás**.

> [!NOTE]
> En el plan gratuito, el servidor se duerme tras 15 minutos sin visitas. La primera carga
> después puede tardar alrededor de un minuto; las siguientes son inmediatas.

## 6. Instálala en los móviles

1. En el ordenador, panel de los papás → **Economía → Acceso → Copiar enlace**. Es la dirección
   con el código dentro (`https://<tu-servicio>.onrender.com/#casa=…`). El fragmento `#casa=`
   nunca llega al servidor ni a sus registros.
2. Mándalo una vez a cada móvil (WhatsApp, AirDrop…) y ábrelo.
3. Instala la app:
   - **Android (Chrome):** menú **⋮ → Instalar app**.
   - **iPhone (Safari):** **Compartir → Añadir a pantalla de inicio**. En iPhone los avisos solo
     funcionan con la app instalada así.
4. Cada uno elige su perfil y activa los avisos (**Cambiar → Avisos en este móvil**).

Guía completa para la familia en el [manual de uso](MANUAL-DE-USO.md).

## 7. Opcional: recordatorio diario y copia nocturna

Render gratis no puede despertarse solo a una hora, así que dos tareas las lanza GitHub Actions
desde tu copia. Están **desactivadas** hasta que configures la dirección:

1. Tu repositorio → **Settings → Secrets and variables → Actions**.
2. Pestaña **Variables** → **New repository variable**: `CASA_URL` =
   `https://<tu-servicio>.onrender.com` (sin barra final).
3. Pestaña **Secrets** → **New repository secret**: `CODIGO_CASA` = el mismo código que en Render.
4. Pruébalas en **Actions → Run workflow**.

| Workflow | Qué hace | Requisito |
|---|---|---|
| `recordatorios.yml` | Entre 17:00 y 21:30 avisa a cada hijo de lo pendiente y a los padres de lo que espera validación | `CASA_URL` |
| `copia-diaria.yml` | Guarda cada noche el estado en la rama `copias` | `CASA_URL` y **repositorio privado** |

> [!CAUTION]
> La copia nocturna contiene los datos reales de tu casa. Si el repositorio es público, el
> workflow **se niega a ejecutarse**. El servidor guarda además una copia diaria en la base de
> datos, que se restaura desde el panel (Economía → Copias de seguridad).

## Actualizar

- **Tus cambios:** cada push a `main` de tu copia despliega solo (`autoDeployTrigger: commit`).
- **Mejoras de este repositorio:**

  ```bash
  git remote add upstream https://github.com/hugorenrisal/casa.git
  git fetch upstream
  git merge upstream/main
  git push
  ```

Las migraciones de base de datos son idempotentes y se aplican al arrancar.

## Límites del plan gratuito

| Límite | Efecto en Casa |
|---|---|
| El servicio se duerme tras 15 min sin tráfico | Primera carga de ~1 min; los ciclos de semana y mes se aplican igualmente en la siguiente lectura |
| 750 horas de instancia al mes por cuenta | Suficiente para un servicio encendido todo el mes |
| Sin disco persistente | No afecta: todo vive en PostgreSQL |
| PostgreSQL de Render caduca a los 30 días | Por eso la guía usa Neon |

Detalle actualizado en la [documentación de Render](https://render.com/docs/free).

## Problemas frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| La construcción falla en `npm ci` | `package-lock.json` desincronizado | En local: `npm install`, commit del lock y push |
| `/api/health` responde `"db": false` | `DATABASE_URL` vacía o incompleta | Render → Environment → pega de nuevo la cadena de Neon → *Save, rebuild and deploy* |
| Los registros dicen `password authentication failed` | Contraseña de Neon rotada o mal copiada | Copia otra vez la cadena desde Neon → Connect |
| La app no acepta el código | No coincide con `CODIGO_CASA` | Revisa la variable en Render; tras 30 intentos fallidos en 15 min esa conexión queda bloqueada un rato (`429`) |
| Tarda mucho en abrir | El servicio estaba dormido | Normal en el plan gratuito; espera un minuto |
| No llegan los avisos en iPhone | La app se abre desde Safari, no instalada | Instálala con *Añadir a pantalla de inicio* y activa los avisos desde ella |
| El workflow de copia falla con «repositorio público» | Protección de datos | Haz el repositorio privado o borra la variable `CASA_URL` si no quieres la copia |
| El workflow falla con `401` | Falta el secreto `CODIGO_CASA` en GitHub o no coincide | Ponlo igual que en Render |

¿Otro problema? Abre un [issue](https://github.com/hugorenrisal/casa/issues/new/choose) sin datos
personales.
