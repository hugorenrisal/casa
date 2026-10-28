# ADR-0003 · Perfiles fijos en lugar de cuentas de usuario

**Estado:** aceptada · sustituye al modelo de cuentas, familias e invitaciones por correo (v1–v2)

## Contexto

La primera versión tenía registro, contraseñas, verificación por correo, familias e
invitaciones. Para una única casa, todo ello añadía fricción (un niño no recuerda contraseñas),
dependencias (proveedor de correo, tokens) y superficie de ataque, sin aportar nada: los miembros
de la casa no cambian.

## Decisión

- Cuatro **perfiles fijos** definidos en `public/perfiles.js`, un módulo UMD que cargan el
  servidor y el navegador: una sola definición, imposible que diverjan.
- Ids **cortos y estables**, referenciados en todo el documento.
- Elegir perfil **no autentica**. El control de acceso es un asunto aparte
  ([ADR-0010](0010-codigo-de-casa-opcional.md)); los perfiles solo determinan qué reglas de
  producto se aplican, y el servidor las hace cumplir (`applyChildLimits`, `soloPapas`).
- Migración: `adoptarPerfiles()` traduce los UUID antiguos a ids de perfil.

## Alternativas

- **Mantener cuentas**: complejidad sin beneficio para un solo hogar.
- **PIN por perfil**: fricción para los hijos; las reglas que importan ya están en el servidor.

## Consecuencias

- ✅ Se eliminaron autenticación, correo, sesiones y tokens.
- ✅ Entrar en la app es un toque.
- ⚠️ Dentro de la casa cualquiera puede usar cualquier perfil. Se acepta: es un problema de
  convivencia, no técnico.
- ⚠️ Cambiar un id de perfil exige migrar el documento.
