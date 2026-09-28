# BankTrack frontend

Dashboard de finanzas personales en React, Vite, TanStack Query y Recharts.
Incluye resumen por período, movimientos editables, cuentas bancarias y chat.
El selector de tema guarda la preferencia local y usa el tema del sistema por defecto.

## Desarrollo

Requiere Node.js 22.12+ y pnpm 11.5.3. Desde este directorio:

```bash
pnpm install --frozen-lockfile
pnpm dev
```

La API se consulta en `http://localhost:8000`. Puedes cambiarla con `VITE_API_URL`.
Consulta el README raíz para iniciar PostgreSQL y el backend.

## Validación y compilación

```bash
pnpm lint
pnpm build
pnpm preview
```

El resultado se genera en `dist/`. El Dockerfile ofrece las etapas
`development` para Vite y `production` para servir la compilación con Nginx.

## Estructura

- `src/pages/Dashboard.jsx`: navegación y composición del panel.
- `src/components/`: resumen, gráficos, movimientos, bancos, chat y tema.
- `src/api/client.js`: llamadas a la API.
- `src/styles/app.css`: estilos adaptables y paleta clara/oscura.
- `public/theme.js`: aplica el tema antes del primer render.
