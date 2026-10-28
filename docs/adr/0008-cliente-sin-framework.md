# ADR-0008 · Cliente sin framework y dependencias mínimas

**Estado:** aceptada · revisable

## Contexto

El cliente empezó como un prototipo de un solo archivo. El proyecto lo mantiene una persona con
ayuda de un agente, se despliega sin paso de compilación y debe funcionar como PWA en móviles
modestos.

## Decisión

- Cliente en HTML, CSS (con *tokens*) y JavaScript sin framework ni *bundler*.
- Servidor con cuatro dependencias de producción: `express`, `pg`, `dotenv`, `web-push`. Ninguna
  de desarrollo: las pruebas usan `node:test`.

## Consecuencias

- ✅ Sin paso de build; despliegue inmediato.
- ✅ Superficie de dependencias mínima y auditable.
- ✅ El script del cliente se prueba tal cual desde Node (`test/_cliente.js`).
- ⚠️ `index.html` ronda las 2.700 líneas. **Revisión prevista:** dividir en módulos ES, sin
  necesidad de adoptar un framework.
