# skapdal/.github

Configuración compartida para los repos de la organización.

## Patrón de build: CI builda, el servidor solo hace pull

Los servidores de producción (Dokploy) **no** buildean imágenes — eso lo hace GitHub Actions, que pushea la imagen ya construida a GHCR (`ghcr.io/skapdal/<repo>`). Dokploy se configura como tipo **"Docker Image"** apuntando a esa imagen, en vez de "build from git".

### Cómo usarlo en un repo con Dockerfile

Agregar `.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    uses: skapdal/shared-workflows/.github/workflows/docker-build-push.yml@main
    with:
      image-name: nombre-del-repo
    secrets: inherit
```

Tags generados automáticamente: el `sha` corto de cada commit, y `latest` en la rama por defecto.

### Redeploy automático (opcional)

Si el repo en Dokploy tiene un webhook de redeploy configurado, agregar ese secreto en el repo de GitHub (`Settings → Secrets and variables → Actions`):

```
DOKPLOY_WEBHOOK_URL = <url del webhook de Dokploy>
```

El workflow lo dispara automáticamente después de cada push exitoso a GHCR. Si no está seteado, el paso se saltea sin fallar el build.

### Repos que no necesitan esto

Repos cuyo `docker-compose.yml` solo usa imágenes públicas (sin `build:` propio) no necesitan este workflow — Dokploy hace `pull` directo, no hay nada que buildear. Ejemplo: `botpress-chatbot`.
