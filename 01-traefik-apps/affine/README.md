# AFFiNE

- URL: https://affine.joseluna.dev
- Contenedores: affine (server), affine-migration, affine-postgres, affine-redis, affine-indexer
- Traefik: `00-proxies/traefik/dynamic/affine.yml` (red `proxy`, entrypoint `https`, resolver `cloudflare`)
- Sablier: grupo `affine` (sesión 1h); los contenedores se apagan solos y Traefik los despierta al entrar.
- Datos: `/mnt/media/alpine-app-copies-20261026/affine/data` (enlace `./data`, ignorado por Git). Comprobar que `/mnt/media` esté montado antes de arrancar.

## Uso

```sh
cp .env.example .env && chmod 600 .env   # solo en una instalación nueva
docker compose config --quiet
docker compose up -d
docker compose logs -f --tail 100
```

`.env` es privado (0600, fuera de Git); `.env.example` es la plantilla sin secretos.
No usar `docker compose down -v`.
