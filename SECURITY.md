# Política de seguridad

## Versiones con soporte

| Versión | Soporte |
|---|:-:|
| 3.3.x | ✅ |
| < 3.3 | ❌ |

## Notificar una vulnerabilidad

No abras un *issue* público. Usa **GitHub → Security → Report a vulnerability** (aviso privado)
en este repositorio. Se responde en un plazo de 7 días.

## Modelo de amenazas

Casa está diseñada para **una sola casa de confianza**. El modelo distingue dos fronteras:

| Frontera | Amenaza | Control |
|---|---|---|
| **Fuera ↔ casa** | Alguien ajeno lee o borra los datos | Código de casa (`X-Casa`), límite de intentos, `noindex` |
| **Dentro de la casa** | Un hijo se aprueba sus tareas o modifica las de otro | Reglas por perfil aplicadas en el servidor |

**Fuera de alcance:** un miembro de la casa que suplanta otro perfil. Es una decisión de
producto documentada en [ADR-0003](docs/adr/0003-perfiles-fijos-sin-cuentas.md).

## Controles implementados

| Control | Implementación |
|---|---|
| Acceso | Código de casa opcional; comparación en tiempo constante; 30 fallos por IP en 15 min → `429` |
| Autorización | `applyChildLimits()` y `soloPapas` en el servidor, independientes del cliente |
| Validación de entrada | `sanitizeState()` por lista blanca: tipos, longitudes, enumerados, referencias; cuerpo ≤ 4 MB |
| Integridad | Concurrencia optimista; rachas e historial calculados solo en el servidor |
| XSS | Escapado sistemático de todo dato antes de pintarlo |
| SSRF | Suscripciones push restringidas a servicios conocidos |
| Cabeceras | `nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: no-referrer`, HSTS, `X-Robots-Tag: noindex`; sin `X-Powered-By` |
| Errores | Las respuestas `500` no incluyen detalles internos |
| Secretos | Solo en variables de entorno; `.env` y claves de firma excluidos de git |
| Dependencias | Dependabot (alertas y correcciones automáticas) y `npm audit --audit-level=high` en CI |
| Recuperación | Copias en tres capas; toda operación destructiva guarda antes una copia |

## Configuración recomendada

- **Activa `CODIGO_CASA`** en cualquier despliegue accesible desde Internet. Sin él, cualquiera
  que conozca la URL puede leer y modificar los datos. `render.yaml` lo pide al crear el servicio.
- La copia nocturna en la rama `copias` contiene los datos reales de la casa: el workflow **se
  niega a ejecutarse en repositorios públicos**.
- No publiques la URL de tu despliegue junto al código ni en issues.
- Guía de instalación segura: [docs/DESPLIEGUE-EN-RENDER.md](docs/DESPLIEGUE-EN-RENDER.md).

## Limitaciones conocidas

- El código de casa es un secreto compartido: no identifica personas ni permite revocar un
  dispositivo concreto.
- No hay registro de auditoría por persona.
- Sin política de seguridad de contenido (CSP), porque el cliente usa scripts en línea.
