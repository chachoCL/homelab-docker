# Copia Alpine → Debian: affine

Datos privados: `/mnt/media/alpine-app-copies-20261026/affine/data` (HDD `/mnt/media`).
Backups: `/mnt/media/alpine-app-copies-20261026/affine/backups`. Entorno: `.env`, modo 0600, ignorado.
Compose: `docker compose -f compose.migration.yaml up -d`.
No usar el compose legado sin `-f`; no mezclar ambos proyectos.

Imágenes fijadas al digest del contenedor realmente desplegado en Alpine; sin upgrade.
Red interna propia por proyecto; servicios web en red existente `proxy`, pero
`traefik.enable=false` (routers explícitos del file provider). Se conservan los
puertos de prueba en 127.0.0.1. Rutas HTTPS en Traefik de Debian, sin cambio de DNS,
Newt, URLs públicas ni del host Alpine.

Servicios de prueba:
- server: http://127.0.0.1:18440/

Las URLs públicas originales se conservaron: redirects, OAuth, callbacks y enlaces
pueden regresar a Alpine. No se ha efectuado corte ni prueba autenticada de usuario.
No se probaron escrituras/borrados S3 ni cambios en buckets; se conservaron sus
credenciales/configuración. La copia puede quedar desactualizada mientras Alpine
sigue recibiendo escrituras. Antes del corte se requiere sincronización final
coordinada y comprobación de autenticación/adjuntos con el usuario.

PostgreSQL activo: dump consistente/restauración (LiteLLM y Blinko), no copia de
PGDATA activo. PostgreSQL detenido: copia física preservando permisos/propietarios.
Blinko: adjuntos verificados por SHA-256 y recuentos por tabla comparados.
Excalidash: backup SQLite online con better-sqlite3 e integrity_check=ok.
Redis sin bind en origen: se preservó /data; LiteLLM por RDB online, AFFiNE y
Outline por copia del contenedor detenido. Datos Redis ahora persistentes en HDD.
Copias antiguas de LiteLLM conservadas sin montarlas.

Fallo previo conocido: Excalidash /health responde 302 en ambos hosts y el
healthcheck espera 200; por ello el backend aparece unhealthy, no es regresión.
El job AFFiNE termina con código 0 y restart=no (no es un daemon).

No ejecutar down -v ni eliminar datos del origen. No commit/push realizado.

## Reverse proxy Debian

Dominio: `https://affine.joseluna.dev`. Archivo: `00-proxies/traefik/dynamic/affine.yml`.
EntryPoint `https`, TLS resolver `cloudflare`; backend `http://copy-affine-server:3010`.
Sablier Debian: grupo exclusivo `copy-affine`, sesión `1h`, incluyendo dependencias del stack.
El job `affine_migration` queda excluido de Sablier para no repetir migraciones al despertar.
Prueba sin cambiar DNS ni seguir redirects: `curl --resolve affine.joseluna.dev:443:10.0.0.6 https://affine.joseluna.dev/`.
Un redirect no demuestra login ni callbacks: no se ha validado autenticación ni escrituras externas.
