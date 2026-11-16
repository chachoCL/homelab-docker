# Copia Alpine → Debian: blinko

Datos privados: `/mnt/media/alpine-app-copies-20261026/blinko/data` (HDD `/mnt/media`).
Backups: `/mnt/media/alpine-app-copies-20261026/blinko/backups`. Entorno: `.env`, modo 0600, ignorado.
Compose predeterminado completo: `compose.yaml`. Desde este directorio:
```sh
docker compose config --quiet
docker compose up -d
```
`compose.migration.yaml` se conserva como archivo anterior, con la misma configuración
y el mismo nombre de proyecto; no usar ambos simultáneamente ni combinarlos con `-f`.
`data` es solo un enlace de navegación al directorio del HDD indicado arriba.
Los bind mounts mantienen sus rutas absolutas en `/mnt/media`; no se movieron
ni copiaron datos al filesystem de `/root`. Conservar montado el HDD antes de arrancar.

Imágenes fijadas al digest del contenedor realmente desplegado en Alpine; sin upgrade.
Red interna propia por proyecto; servicios web en red existente `proxy`, pero
`traefik.enable=false` (routers explícitos del file provider). Se conservan los
puertos de prueba en 127.0.0.1. Rutas HTTPS en Traefik de Debian, sin cambio de DNS,
Newt, URLs públicas ni del host Alpine.

Servicios de prueba:
- blinko-website: http://127.0.0.1:18420/

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

Dominio: `https://keep.joseluna.dev`. Archivo: `00-proxies/traefik/dynamic/blinko.yml`.
EntryPoint `https`, TLS resolver `cloudflare`; backend `http://copy-blinko-blinko-website:1111`.
Sin middleware/etiquetas Sablier, igual que en Caddy origen: siempre activo.
Prueba sin cambiar DNS ni seguir redirects: `curl --resolve keep.joseluna.dev:443:10.0.0.6 https://keep.joseluna.dev/`.
Un redirect no demuestra login ni callbacks: no se ha validado autenticación ni escrituras externas.
