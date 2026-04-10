# Homelab — moon-stack

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

- Los datos persistentes se almacenan en volúmenes o carpetas `*_data` junto a cada servicio (ej. `excalidraw/mongodb`, `evolution-api/postgres_data`).
- Implementa backups regulares de los volúmenes y bases de datos (cron + `docker run --rm -v ...`).

## Seguridad y mantenimiento

- Mantén Docker y las imágenes actualizadas.
- Revisa usuarios y credenciales en cada `env.example` y usa secretos cuando sea posible.
- Habilita firewall y acceso SSH seguro (clave pública, 2FA donde aplique).

## Contribuir / Cambios

- Añade nuevos servicios creando una carpeta con su `compose.yaml` y, si aplica, un `env.example`.
- Abre un PR para cambios que afecten a otros servicios o la configuración de proxies.

## Información pendiente / Personalizar

- Dominio principal: **(poner dominio aquí)**
- Contacto/propietario: **(tu nombre / email)**
- Credenciales y secretos: no incluir en el repo; usar `.env` o un gestor de secretos.

---

Si quieres, personalizo este `README` con más detalles: dominios, servicios críticos, procedimientos de backup/restore o instrucciones de arranque automático.
