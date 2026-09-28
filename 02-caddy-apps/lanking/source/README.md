# BankTrack 🏦

Dashboard financiero para bancos chilenos: importa saldos y movimientos mediante
[open-banking-chile](https://github.com/bcortezf/open-banking-chile), categoriza con
IA y permite consultar movimientos, corregir categorías y conversar sobre tus gastos.

## Arquitectura

- **Frontend:** React 19, Vite, TanStack Query, Recharts y Lucide.
- **Backend:** FastAPI, SQLAlchemy asíncrono y PostgreSQL.
- **Conexión bancaria:** proyecto Node/Chromium en `backend/open-banking-chile`.
  FastAPI ejecuta `backend/open_banking_runner.mjs`, que carga su `dist/index.js`.
- **IA:** LangGraph y ChatOpenAI contra un endpoint compatible, como LiteLLM.
  Chat y categorización comparten configuración.

open-banking-chile reemplaza el scraping anterior, no la API ni la base de datos.

## Inicio local recomendado

Requisitos: Docker Compose, uv, Node.js 22.12+, pnpm 11.5.3, npm y Chrome/Chromium.
Ejecuta los comandos desde la raíz del repositorio.

### 1. Configurar

Copia la plantilla **solo si todavía no tienes `.env`**:

```bash
cp -n .env.example .env
```

Configura el `.env` raíz:

```dotenv
DATABASE_URL=postgresql+asyncpg://banktrack:banktrack_dev@localhost:5432/banktrack
LITELLM_PROXY_URL=https://TU-PROXY-COMPATIBLE
OPENAI_API_KEY=TU-TOKEN-VALIDO
AI_MODEL=deepseek-v4-flash
```

Usa un modelo habilitado y un token válido **para ese servidor**. Una clave de
OpenAI no sirve automáticamente como token de un proxy LiteLLM. Para OpenAI directo,
configura su URL compatible y un modelo disponible en tu cuenta.

Las variables exportadas en la terminal tienen prioridad sobre `.env`.
**Tras cambiar `.env`, detén y vuelve a iniciar el backend**: `--reload` no garantiza
recargar esas variables. No hay claves de respaldo hardcodeadas.

### 2. Instalar dependencias y compilar el proveedor

```bash
uv sync
pnpm --dir frontend install --frozen-lockfile
(cd backend/open-banking-chile && npm ci && npm run build)
```

Después de modificar el TypeScript del proveedor, vuelve a ejecutar su
`npm run build`: el backend usa `dist`, no los archivos de `src` directamente.

### 3. Iniciar

```bash
docker compose up -d --wait db
OPEN_BANKING_HEADFUL=true pnpm dev
```

`pnpm dev` desde la raíz inicia API y frontend juntos. No inicia PostgreSQL.
`OPEN_BANKING_HEADFUL=true` muestra el navegador durante la conexión bancaria;
requiere una sesión gráfica local. Omítelo para ejecución sin navegador visible.
Si no se detecta Chrome, configura `CHROME_PATH` con la ruta del ejecutable.

También puedes usar dos terminales en lugar de `pnpm dev`:

```bash
# Terminal 1, desde la raíz
OPEN_BANKING_HEADFUL=true uv run uvicorn backend.main:app --reload --log-level info

# Terminal 2, desde la raíz
pnpm --dir frontend dev
```

- **Dashboard:** http://localhost:5173/
- **API y Swagger:** http://localhost:8000/docs
- **Salud:** http://localhost:8000/api/health

### 4. Detener sin perder datos

Con `pnpm dev`, pulsa **Ctrl+C** para detener frontend y backend. Si los ejecutaste
en terminales separadas, detén ambos con Ctrl+C. Después:

```bash
docker compose stop
```

El volumen PostgreSQL conserva tus datos. **No uses `docker compose down -v`**
si quieres conservarlos.

## Configuración de IA desde la app

Abre **http://localhost:5173/config** o «Configuración de IA» en el menú.
Selecciona OpenAI, OpenRouter, Anthropic (API nativa) o Custom compatible con
OpenAI; indica el modelo, URL y API key. Los proveedores conocidos tienen URL fija;
para otra URL utiliza Custom.

Los ajustes se guardan en `backend/.private/ai.json` con permisos `0600`, fuera de
Git y de la imagen Docker. El archivo contiene la clave en texto plano: protege
el servidor y sus copias de seguridad. La API no devuelve la clave y el navegador
no la persiste. Dejarla vacía conserva la existente solo si no cambia el proveedor
ni la URL. Cambiar de destino exige introducir una nueva clave.

Lo guardado tiene prioridad sobre `.env` y se aplica a la próxima llamada sin
reiniciar. Guardar valida la configuración, **no comprueba la conectividad ni la
validez del token**. Para volver a `.env`, elimina manualmente ese archivo privado.

Por seguridad, mientras no exista autenticación, estos endpoints solo aceptan
acceso directo desde localhost y los orígenes locales de desarrollo. El acceso
mediante Docker bridge, proxy o desde otro equipo se rechaza; usa el backend local
con uv para configurar desde esta pantalla. No expongas la app sin autenticación.

## Sincronizar Santander y ver logs

1. Abre el dashboard, pulsa **+ Banco** y conecta Santander con tus credenciales.
2. Atiende cualquier aprobación que solicite el banco en el navegador.
3. Observa la terminal del backend: primero se recorren todas las cuentas
   **Corriente/Vista**, después las tarjetas de crédito.
4. Espera el resultado de la sincronización. **Si las categorías o movimientos no
   aparecen actualizados, recarga la página**; la UI puede seguir mostrando datos
   anteriores aunque el proceso haya terminado correctamente.

Ejemplo de eventos (la cantidad de cuentas depende de tu sesión):

```text
[openbanking:santander] accounts_start
[openbanking:santander] account_start_1_of_3
[openbanking:santander] account_complete_1_of_3
...
[openbanking:santander] accounts_complete
[openbanking:santander] credit_cards_start
[openbanking:santander] credit_cards_complete
[openbanking:santander] bank_complete
[sync:santander:ID] received ... movements
[sync:santander:ID] saved ... new movements; categorizing pending movements
AI configuration: model=deepseek-v4-flash base_url=...
[sync:santander:ID] finished status=ok
```

Los tokens `progress_N`/`activity_N` indican actividad, no necesariamente una etapa
completada. Los logs del adaptador no vuelcan el JSON bancario ni los mensajes
crudos de debug del proveedor.

### Resultados y seguridad de los datos

- **`ok`:** sincronización finalizada correctamente.
- **`partial`:** se guardaron movimientos, pero falló la categorización. Revisa la
  configuración de IA, reinicia y repite la sincronización para reintentar pendientes.
- **`error`:** revisa la etapa indicada; no equivale necesariamente a un fallo de IA.

Santander devuelve saldos y movimientos separados por cuenta en el resultado del
scraper, y tarjetas en otro bloque. El dashboard sigue consolidando los datos por
banco. Si una cuenta no se puede activar o no se puede verificar su saldo en su
propio elemento del carrusel, el scraper falla antes de tarjetas: no importa un
saldo tomado de otra cuenta ni presenta esa lectura como completa.

Los identificadores de movimientos Santander distinguen cuentas diferentes.
**Compatibilidad con importaciones antiguas:** cuando un identificador anterior
corresponde ambiguamente a movimientos de varias cuentas, se conserva la fila
antigua y se registran las nuevas con una advertencia. Puede quedar un duplicado
histórico; no se borran ni fusionan datos financieros por adivinación.

Para consultar el resultado persistido sin mostrar movimientos ni credenciales:

```bash
docker compose exec db psql -U banktrack -d banktrack -c \
  'SELECT id, account_id, status, transactions_found, transactions_new, error_message FROM sync_logs ORDER BY id DESC LIMIT 10;'
```

Los logs locales van a la terminal. Si rediriges la salida a un archivo, puedes
seguirlo con `tail -f RUTA`. `/tmp/lanking-logs/backend.log` fue usado en una sesión
manual; no es un archivo creado automáticamente por `pnpm dev`.

## Pruebas

Sin credenciales bancarias ni llamadas reales a IA:

```bash
uv run python -m backend.check_ai_config
uv run python -m unittest backend.test_open_banking backend.test_sync_identity
(cd backend/open-banking-chile && npm test)
```

Con PostgreSQL configurado y levantado:

```bash
uv run python -m backend.check_api
uv run python -m backend.check_sync
```

`check_api` inicia el ciclo de vida real de FastAPI y crea tablas faltantes.
`check_sync` usa banco e IA simulados con PostgreSQL real, verifica importación,
deduplicación y recuperación de un fallo de IA; limpia sus propias filas de prueba.
Las pruebas bancarias E2E dependen de credenciales: no las confundas con las pruebas
simuladas. No compartas salidas con datos financieros sin censurarlas.

Frontend:

```bash
pnpm --dir frontend lint
pnpm --dir frontend build
```

**Validación realizada:** el recorrido real de Santander completó tres cuentas
antes de tarjetas y terminó la categorización con `deepseek-v4-flash`. Esto no
garantiza otros bancos, todas las variantes de cuenta ni futuros cambios del portal.

## Ejecutar todo con Docker Compose

Configura primero `.env`, incluida una `LITELLM_PROXY_URL` accesible desde el
contenedor. `localhost` dentro del backend es el propio contenedor, no tu host.

```bash
docker compose up -d --build
docker compose logs -f backend
# Para detener, conservando datos:
docker compose stop
```

Las URLs públicas son las mismas que en desarrollo local. El navegador visible
local es la opción recomendada para depurar aprobaciones manuales; la configuración
Compose no comparte automáticamente tu sesión gráfica.

El Compose actual es de desarrollo (bind mounts y recarga), no un despliegue
endurecido de producción. No existe un `docker-compose.prod.yml` en este repositorio.

## Bancos del proveedor

Banco de Chile (`bchile`), Santander (`santander`), BancoEstado (`bestado`),
BCI (`bci`), BICE (`bice`), Banco Edwards (`edwards`), Falabella (`falabella`),
Banco Security (`bancosecurity`), Cencosud (`cencosud`), Itaú (`itau`) y
Scotiabank (`scotiabank`).

La disponibilidad del conector no implica una prueba real de cada banco.
El banco puede exigir 2FA, CAPTCHA o aprobación manual.

## Configuración principal

| Variable | Uso |
| --- | --- |
| `DATABASE_URL` | Conexión PostgreSQL; el valor local por defecto coincide con Compose. |
| `OPENAI_API_KEY` | Token del servidor de IA configurado; obligatorio para las funciones IA. |
| `LITELLM_PROXY_URL` | URL compatible con OpenAI para chat y categorización. Configúrala explícitamente. |
| `AI_MODEL` | Modelo compartido; por defecto `deepseek-v4-flash`. |
| `CHROME_PATH` | Ruta de Chrome/Chromium para la conexión bancaria. |
| `OPEN_BANKING_HEADFUL` | `true` para mostrar el navegador local. |
| `OPEN_BANKING_XVFB` | `true` para envolver el proceso con `xvfb-run` donde esté instalado. |
| `OPEN_BANKING_DIR` | Directorio alternativo del proveedor compilado. |
| `VITE_API_URL` | URL del backend para el frontend. |

## API

Documentación interactiva completa en `/docs`. Rutas principales:

- `POST /api/sync`
- `GET /api/accounts`
- `GET /api/transactions`
- `PATCH /api/transactions/{transaction_id}/category`
- `PATCH /api/transactions/{transaction_id}/merchant`
- `GET /api/summary`
- `POST /api/chat`

## Problemas frecuentes

**`ConnectionRefusedError` al arrancar:** levanta PostgreSQL con
`docker compose up -d --wait db` y revisa `DATABASE_URL`.

**401 de LiteLLM:** el token no es válido para ese proxy. No es un fallo del login
bancario. Revisa `OPENAI_API_KEY` y `LITELLM_PROXY_URL` sin publicar sus secretos.

**Sigue usando otro modelo:** comprueba `AI_MODEL`, variables exportadas en la
terminal y el log de configuración efectiva. Reinicia el backend por completo.

**Sin categorías visibles pero el log termina en `ok`:** recarga el dashboard.
Si termina en `partial`, corrige la IA y reintenta la sincronización.

**Cambios del scraper no aparecen:** recompila `backend/open-banking-chile`.

**Fallo bancario:** comprueba las etapas en logs, el navegador, credenciales y
aprobaciones pendientes. No reintentes indefinidamente: el banco puede bloquear
intentos repetidos. El portal puede haber cambiado y requerir ajustar el conector.

## Seguridad

Las credenciales bancarias se pasan al proceso local mediante su entorno y no se
persisten en la base de datos. El proceso del proveedor recibe un entorno acotado,
sin las claves de IA del backend. No compartas `.env`, capturas, HTML ni respuestas
bancarias: pueden contener información sensible.

Esta configuración está orientada al uso local. Para exponerla en producción,
revisa autenticación/autorización, HTTPS, CORS, protección de PostgreSQL y gestión
de secretos; no asumas que el Compose de desarrollo cubre esas medidas.

## Créditos

Conectividad bancaria basada en [open-banking-chile](https://github.com/bcortezf/open-banking-chile).
Consulta `backend/open-banking-chile/UPSTREAM.md` para notas del proveedor y actualización.
