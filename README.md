# homelab

Infraestructura compartida de mis apps personales: el servidor Proxmox, el login central (IdP),
el proxy con HTTPS, la red (Tailscale) y las copias de seguridad.

Las apps (ContApp y las que vengan) viven en sus propios repos. Aquí solo está lo que comparten
y cómo se despliegan. Lo que se va haciendo, y lo aprendido, está en el
[registro](docs/registro.md).

## Cómo encaja todo

```
                ¿llegas a la red?
Navegador ──► Tailscale ──► Proxy (HTTPS) ──┬──► App (ContApp…)        ¿qué datos son tuyos?
                                            │         │
                                            │         │ back-channel: canjea el código por el token
                                            │         ▼
                                            └──► IdP (login OIDC)      ¿quién eres y puedes abrir la app?
```

1. El navegador llega por Tailscale al proxy, que reparte el tráfico según el nombre: a la app
   o al IdP.
2. Si no tienes sesión, la app te redirige al IdP. Inicias sesión allí y el IdP te devuelve a
   la app con un código de un solo uso.
3. La app canjea ese código por el token directamente con el IdP (back-channel, sin pasar por
   el navegador) y crea su propia sesión.

- **Tailscale** decide quién llega a la red, y a qué máquinas.
- **El IdP** dice quién eres y si perteneces al grupo de una app (por ejemplo `contapp`).
  Las apps nunca ven contraseñas.
- **Cada app** decide qué datos ves, con sus propias tablas de permisos.

**OIDC frente a forward auth:** mis apps hablan OIDC con el IdP y gestionan su propia sesión;
el proxy no interviene en el login. El forward auth (el proxy pregunta al IdP en cada petición
antes de dejar pasar) queda para apps de terceros que no tienen login propio.

## Inventario

Red de casa `192.168.1.0/24`: IPs fijas en `.2`–`.99` y DHCP en `.100`–`.199`.
Dominio local: `majemase.internal`.

| Máquina | Función | IP | Estado |
|---|---|---|---|
| `pve.majemase.internal` | Proxmox VE (instalado sobre ext4) | 192.168.1.97 | Instalado |
| LXC de Tailscale (no privilegiado) | Subnet router de toda la red de casa | 192.168.1.98 | Funcionando |
| LXC de BD | Bases de datos | — | Pendiente |
| LXC de apps | Apps en Docker (con nesting) | — | Pendiente |
| LXC del proxy | Proxy inverso con nombres y HTTPS | — | Pendiente |
| LXC del IdP | Login central (OIDC) | — | Pendiente |

## Estructura

| Carpeta | Contenido | Estado |
|---|---|---|
| `docs/adr/` | Decisiones de arquitectura (una por archivo) | En marcha |
| `docs/registro.md` | Bitácora: qué se hizo y qué aprendí | En marcha |
| `idp/` | IdP: `compose.yaml`, alta de clientes y configuración. Solo se sube una configuración de ejemplo con datos inventados | Pendiente de elegir (Pocket ID o Authelia) |
| `proxy/` | Proxy inverso con nombres y HTTPS | Pendiente |
| `apps/<app>/` | Despliegue de cada app (`compose.yaml`, `.env.example`) | Pendiente |
| `tailscale/` | Política de acceso de la tailnet (grants) con sus tests. Solo se sube una versión de ejemplo con datos inventados | Tailscale funcionando; pendiente la política de acceso y sus tests |
| `backups/` | Copias programadas y cómo restaurarlas | Pendiente |

## Normas

- **Los secretos nunca van a git.** Cada servicio tiene un `.env.example` (sí se sube) que
  documenta las variables, y un `.env` (no se sube) con los valores reales.
- **Los datos personales y los identificadores de la tailnet tampoco.** Ni emails de usuarios,
  ni grupos con personas reales, ni auth keys, ni el nombre de la tailnet. De la política de
  Tailscale y de la configuración del IdP se sube solo una versión de ejemplo con datos
  inventados, igual que un `.env.example`.
- **El Dockerfile de cada app vive en el repo de la app**: la app sabe cómo construirse.
  Aquí se decide dónde y cómo se despliega.
- **Las apps solo hablan OIDC.** No dependen de qué IdP haya detrás: cambiarlo o añadir una
  app nueva consiste en registrar un cliente en el IdP y rellenar su `.env`.
- **Cada decisión importante, un ADR** en `docs/adr/` (plantilla en
  [docs/adr/README.md](docs/adr/README.md)).
- **Una copia que nunca has restaurado no es una copia.** Cada servicio con datos documenta
  cómo se restaura, y se prueba.

## Añadir una app nueva al login

1. En el IdP: crear el grupo de la app y registrar **dos clientes OIDC**, uno por entorno, con
   el acceso restringido a ese grupo:
   - **desarrollo** (por ejemplo `contapp-dev`), con la URL de retorno en `localhost`;
   - **producción** (por ejemplo `contapp`), con la URL de retorno del dominio real.

   Así un secreto de desarrollo nunca vale en producción, y el de producción no sale del
   servidor.
2. En la app: rellenar en el `.env` de cada entorno `OIDC_ISSUER`, `OIDC_CLIENT_ID`,
   `OIDC_CLIENT_SECRET` y `OIDC_REDIRECT_URI` con los datos de su cliente.
3. Añadir a los usuarios al grupo.

(Se concretará con el IdP elegido.)
