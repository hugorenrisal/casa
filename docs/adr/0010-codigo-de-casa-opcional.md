# ADR-0010 · Código de casa opcional como control de acceso

**Estado:** aceptada

## Contexto

Sin cuentas ([ADR-0003](0003-perfiles-fijos-sin-cuentas.md)), cualquiera que conociera la URL
podía leer o borrar el estado. Pero pedir credenciales a cada miembro devolvía la fricción que se
había eliminado.

## Decisión

- Un **código de casa** (`XXXX-XXXX-XXXX`: 12 caracteres aleatorios de un alfabeto sin caracteres
  ambiguos) en la variable `CODIGO_CASA`.
- Cada dispositivo lo recibe **una vez** por enlace (`#casa=…`; el fragmento no sale del
  navegador) o escribiéndolo, y lo envía en la cabecera `X-Casa`.
- Comparación en tiempo constante sobre huellas SHA-256; **30 fallos por IP en 15 minutos**
  bloquean esa IP. Una petición sin código no consume intentos.
- **Opcional**: sin la variable, la API es abierta. Es la configuración elegida mientras la app
  sea de uso privado, y se activa sin tocar código.

## Consecuencias

- ✅ Separa acceso (`X-Casa`) de reglas de producto (`X-Perfil`).
- ✅ Activable en minutos ([despliegue](../DESPLIEGUE.md#activar-el-código-de-casa)).
- ⚠️ Es un secreto compartido: no distingue personas ni permite revocar un dispositivo concreto
  (hay que rotar el código).
