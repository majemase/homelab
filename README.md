# homelab

Infraestructura compartida de mis apps personales: el servidor Proxmox, el login central (IdP),
el proxy con HTTPS, la red (Tailscale) y las copias de seguridad.

Las apps (ContApp y las que vengan) viven en sus propios repos. Aquí solo está lo que comparten
y cómo se despliegan.

## Cómo encaja todo

```
Navegador ──► Tailscale ──► Proxy (HTTPS) ──► App (ContApp…)
                 │                │               │
           ¿llegas a la red?      │          ¿qué datos son tuyos?
                                  ▼
                          IdP (login OIDC)
                    ¿quién eres y puedes abrir la app?
```

- **Tailscale** decide quién llega a la red, y a qué máquinas.
- **El IdP** dice quién eres y si perteneces al grupo de una app (por ejemplo `contapp`).
  Las apps nunca ven contraseñas.
- **Cada app** decide qué datos ves, con sus propias tablas de permisos.

## Estructura

| Carpeta | Contenido | Estado |
|---|---|---|
| `docs/adr/` | Decisiones de arquitectura (una por archivo) | En marcha |
| `idp/` | IdP: `compose.yaml`, configuración, alta de clientes | Pendiente de elegir (Pocket ID o Authelia) |
| `proxy/` | Proxy inverso con nombres y HTTPS | Pendiente |
| `apps/<app>/` | Despliegue de cada app (`compose.yaml`, `.env.example`) | Pendiente |
| `tailscale/` | Política de la tailnet (ACL/grants) con sus tests | Pendiente |
| `backups/` | Copias programadas y cómo restaurarlas | Pendiente |

## Normas

- **Los secretos nunca van a git.** Cada servicio tiene un `.env.example` (sí se sube) que
  documenta las variables, y un `.env` (no se sube) con los valores reales.
- **El Dockerfile de cada app vive en el repo de la app**: la app sabe cómo construirse.
  Aquí se decide dónde y cómo se despliega.
- **Las apps solo hablan OIDC.** No dependen de qué IdP haya detrás: cambiarlo o añadir una
  app nueva consiste en registrar un cliente en el IdP y rellenar su `.env`.
- **Cada decisión importante, un ADR** en `docs/adr/` (plantilla en
  [docs/adr/README.md](docs/adr/README.md)).
- **Una copia que nunca has restaurado no es una copia.** Cada servicio con datos documenta
  cómo se restaura, y se prueba.

## Añadir una app nueva al login

1. En el IdP: crear el grupo de la app y registrar la app como cliente OIDC, con su URL de
   retorno y el acceso restringido a ese grupo.
2. En la app: rellenar en su `.env` `OIDC_ISSUER`, `OIDC_CLIENT_ID`, `OIDC_CLIENT_SECRET` y
   `OIDC_REDIRECT_URI`.
3. Añadir a los usuarios al grupo.

(Se concretará con el IdP elegido.)
