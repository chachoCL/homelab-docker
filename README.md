
Este repositorio contiene la configuración y los stacks Docker/Compose de mi homelab. Está organizado por carpetas por propósito (proxies, apps gestionadas por Traefik/Caddy, servicios compartidos, etc.).

## Descripción

Este árbol contiene despliegues en contenedores para servicios personales, proxies reversos y utilidades de mantenimiento. Está pensado para ejecutarse en una máquina Linux con Docker y Docker Compose V2.

## Estructura principal (resumen)

- `00-proxies/` — configuraciones de reverse proxy (Traefik, Caddy) y certificados.
- `01-traefik-apps/` — aplicaciones gestionadas detrás de Traefik (cada subcarpeta es un servicio con su `compose.yaml`).
- `02-caddy-apps/` — aplicaciones gestionadas con Caddy.
- `shared/` — utilidades y configuraciones compartidas entre stacks.

Cada servicio suele tener un `compose.yaml` dentro de su carpeta. Revisa la carpeta del servicio para instrucciones específicas.

## Servicios importantes

- Proxies: Traefik, Caddy (en `00-proxies/`).
- Plataformas y apps: Portainer, AdGuard, Homebox, UptimeKuma, etc. (ver `01-traefik-apps/` y `02-caddy-apps/`).

## Cómo desplegar

1. Entrar en la carpeta del servicio que quieras levantar, por ejemplo:

```bash
cd 01-traefik-apps/traefik
docker compose up -d
```

2. Para actualizar imágenes y reiniciar:

```bash
docker compose pull
docker compose up -d
```

3. Para ejecutar todos los stacks de una carpeta puedes usar un bucle (ejemplo básico):

```bash
for d in */; do (cd "$d" && docker compose up -d); done
```

Adapta los comandos según la estructura de cada subcarpeta.

## Redes y certificados

- Los proxies centralizan certificados (Let's Encrypt) y enrutamiento. Revisa `00-proxies/traefik` y `00-proxies/caddy` para detalles de certificados y configuración de dominios.

## Backups y datos persistentes

- Los datos persistentes se almacenan en volúmenes o carpetas `data`, `*-data`, etc. junto a cada servicio.
- Implementa backups regulares de los volúmenes y bases de datos (cron + `docker run --rm -v ...`).

## Migrar el homelab a otro host

El repo (`.gitignore` en lista blanca) solo versiona lo reproducible:

1. **En Git:** `compose.yaml`, `env.example`, `Caddyfile`, `traefik/dynamic/`, `00-proxies/traefik/data/traefik.yml`, y `dockerfile`/`Dockerfile` si aplica.
2. **Fuera de Git (restaurar en el destino):** `.env`, `acme.json`, tokens (`cf_api_token.txt`), bases de datos, carpetas `*-data`, cachés de apps.

Flujo típico:

```bash
git clone <este-repo> moon-stack
cd moon-stack
# Por cada servicio: cp env.example .env, editar secretos, luego docker compose up -d
# Restaurar backups de volúmenes/carpetas de datos en las rutas que monta cada compose
```

Levanta primero `00-proxies/` (red `proxy` y certificados) y después las apps.

## Seguridad y mantenimiento

- Mantén Docker y las imágenes actualizadas.
- Revisa usuarios y credenciales en cada `env.example` y usa secretos cuando sea posible.
- Habilita firewall y acceso SSH seguro (clave pública, 2FA donde aplique).

## Contribuir / Cambios

- Añade nuevos servicios creando una carpeta con su `compose.yaml` y, si aplica, un `env.example`.
- Abre un PR para cambios que afecten a otros servicios o la configuración de proxies.

## Contacto

- Dominio principal: [joseluna.dev](https://joseluna.dev)
- Contacto: [git@joseluna.dev](mailto:git@joseluna.dev)
- Credenciales y secretos: no incluir en el repo; usar `.env` o un gestor de secretos.
