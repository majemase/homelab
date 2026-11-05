# 0001. IdP central con OIDC para todas las apps

- Fecha: 2026-11-05
- Estado: Aceptado

## Contexto

Tengo varias apps personales en el homelab (la primera, ContApp) y quiero que las usen otras
personas, cada una con su usuario. Si cada app tiene su propio login, cada una guarda
contraseñas, implementa el alta, la recuperación y el segundo factor, y cada usuario tiene una
cuenta distinta por app.

Recursos limitados: un servidor Proxmox con 8 GB de RAM.

## Decisión

Un **proveedor de identidad (IdP) central**, en su propio LXC, al que todas las apps se
conectan con **OpenID Connect (OIDC)**.

- El IdP autentica (quién eres) y da permiso grueso por **grupos** (por ejemplo, `contapp`
  para poder abrir ContApp).
- Cada app autoriza (qué datos ves) con sus propias tablas. En ContApp, `ledger_members`.
- Las apps identifican al usuario por el claim **`sub`**, que es permanente. El email solo es
  informativo: puede cambiar y nunca se usa para enlazar cuentas.
- Las apps crean al usuario en su primer acceso (alta JIT).
- El IdP vive en este repo (`homelab`), no en el de ninguna app.

IdP concreto: pendiente (Pocket ID o Authelia, ver un ADR posterior).

## Alternativas descartadas

- **Login propio en cada app**: repetir lo más delicado (contraseñas, sesiones, recuperación)
  en cada app, y una cuenta por app para cada usuario.
- **Keycloak / Authentik**: completos, pero demasiado pesados para 8 GB compartidos con el
  resto de servicios.
- **Solo Tailscale como control de acceso**: dice quién llega a la red, no quién es dentro de
  cada app. Sirve como primera capa, no como login.

## Consecuencias

- Un solo usuario y un solo login (SSO) para todas las apps; las apps nunca ven contraseñas.
- Añadir una app nueva consiste en registrar un cliente OIDC y crear su grupo.
- El IdP pasa a ser una pieza crítica: si cae, no se puede entrar en ninguna app. Necesita
  copias de seguridad probadas.
- Exige el proxy con HTTPS y una URL estable: el navegador y las apps deben alcanzar el IdP en
  la misma dirección, y las passkeys no funcionan sin HTTPS fuera de `localhost`.
- En desarrollo, cada app puede funcionar sin IdP con un usuario de desarrollo, que nunca vale
  en producción.
