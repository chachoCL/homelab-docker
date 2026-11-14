# Outline

- URL: https://outline.joseluna.dev
- Contenedores: outline, outline-postgres, outline-redis
- Traefik: `00-proxies/traefik/dynamic/outline.yml` (red `proxy`, entrypoint `https`, resolver `cloudflare`)
- Sablier: grupo `outline` (sesión 1h); los contenedores se apagan solos y Traefik los despierta al entrar.
- Datos: `/mnt/media/alpine-app-copies-20261026/outline/data` (enlace `./data`, ignorado por Git). Comprobar que `/mnt/media` esté montado antes de arrancar.

## Uso

```sh
cp .env.example .env && chmod 600 .env   # solo en una instalación nueva
docker compose config --quiet
docker compose up -d
docker compose logs -f --tail 100
```

`.env` es privado (0600, fuera de Git); `.env.example` es la plantilla sin secretos.
No usar `docker compose down -v`.
