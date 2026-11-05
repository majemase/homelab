# Registro

Bitácora del homelab: qué se hizo y qué aprendí. La entrada más reciente, arriba.

## 2026-11-05 · Repo `homelab` y ADR 0001

**Qué se hizo:** creado este repo para la infraestructura compartida, separado de las apps,
y escrito el primer ADR: un IdP central con OIDC para todas las apps
([0001](adr/0001-idp-central-oidc.md)).

**Qué aprendí:** el IdP no pertenece a ninguna app. Si lo van a usar varias, es
infraestructura compartida como el proxy o Tailscale, y cada app solo guarda su parte de
cliente OIDC.

## TODO (fecha) · Acceso a Proxmox desde el móvil fuera de casa

**Qué se hizo:** acceso a Proxmox desde el móvil, fuera de la red de casa.

**Qué aprendí:** TODO

## TODO (fecha) · LXC de Tailscale como subnet router

**Qué se hizo:** LXC no privilegiado con Tailscale (192.168.1.98), configurado como subnet
router de toda la red de casa (`192.168.1.0/24`).

**Qué aprendí:** TODO

## TODO (fecha) · Instalación de Proxmox

**Qué se hizo:** instalado Proxmox VE sobre ext4 en `pve.majemase.internal` (192.168.1.97).

**Qué aprendí:** TODO
