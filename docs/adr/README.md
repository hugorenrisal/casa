# Registro de decisiones de arquitectura (ADR)

Cada ADR documenta una decisión relevante: su contexto, las alternativas consideradas y sus
consecuencias. Formato de [Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions).

| # | Decisión | Estado |
|---|---|---|
| [0001](0001-documento-unico-de-estado.md) | Un único documento JSON de estado por casa | Aceptada |
| [0002](0002-concurrencia-optimista.md) | Concurrencia optimista con versión y fusión en el cliente | Aceptada |
| [0003](0003-perfiles-fijos-sin-cuentas.md) | Perfiles fijos en lugar de cuentas de usuario | Aceptada (sustituye al modelo de cuentas) |
| [0004](0004-normalizacion-en-el-servidor.md) | Normalizar el estado solo en el servidor | Aceptada |
| [0005](0005-reloj-del-servidor.md) | El reloj del servidor decide los ciclos | Aceptada |
| [0006](0006-web-push-propio.md) | Web Push estándar con VAPID, sin proveedor externo | Aceptada |
| [0007](0007-github-actions-como-planificador.md) | GitHub Actions como planificador de tareas | Aceptada |
| [0008](0008-cliente-sin-framework.md) | Cliente sin framework y dependencias mínimas | Aceptada (revisable) |
| [0009](0009-copias-en-tres-capas.md) | Copias de seguridad en tres capas | Aceptada |
| [0010](0010-codigo-de-casa-opcional.md) | Código de casa opcional como control de acceso | Aceptada |

Para proponer una decisión nueva, copia la estructura de cualquiera de ellas con el siguiente
número libre.
