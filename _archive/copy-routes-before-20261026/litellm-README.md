# Copia Alpine → Debian: litellm

Datos privados: `/mnt/media/alpine-app-copies-20261026/litellm/data` (HDD `/mnt/media`).
Backups: `/mnt/media/alpine-app-copies-20261026/litellm/backups`. Entorno: `.env`, modo 0600, ignorado.
Compose: `docker compose -f compose.migration.yaml up -d`.
No usar el compose legado sin `-f`; no mezclar ambos proyectos.

Imágenes fijadas al digest del contenedor realmente desplegado en Alpine; sin upgrade.
Red interna propia por proyecto; servicios web en red existente `proxy`, pero
`traefik.enable=false`. Solo puertos 127.0.0.1: no se modificaron DNS, Traefik,
Sablier, Newt ni dominios. Acceso de prueba mediante túnel SSH al puerto local.

Servicios de prueba:
- adminer: http://127.0.0.1:18400/
- litellm: http://127.0.0.1:18401/
- prometheus: http://127.0.0.1:18402/

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

Validación adicional: los dos directorios PostgreSQL antiguos se conservaron completos: 1686 y 1689 archivos SHA-256 idénticos. Prometheus se copió en caliente y arranca/responde, pero no es un snapshot transaccional de métricas.
