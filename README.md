# Cashly · Web

Cliente web de **Cashly**, una app que registra tus gastos automáticamente cuando pagas con Apple Pay.
Este repositorio contiene dos cosas:

| Ruta | Qué es | Archivo |
|---|---|---|
| `/` | **Landing page**: explica el producto y tiene el QR para descargar la app móvil | `src/pages/index.astro` |
| `/app` | **La aplicación**: login, panel de gastos, reportes, metas, perfil y panel de administración | `src/pages/app.astro` |

La web no guarda datos propios: todo lo lee y escribe en la **API de Cashly** (repositorio `API-JDS`).

---

## Tecnologías

- [Astro 5](https://astro.build) — genera un sitio **estático** (HTML + JS), sin servidor propio.
- [Tailwind CSS 4](https://tailwindcss.com) — estilos, con la paleta de Cashly convertida en clases.
- [`qrcode`](https://www.npmjs.com/package/qrcode) — genera el QR de descarga **al compilar**.
- Fuente **Inter** (Google Fonts).

## Empezar

Requisitos: Node.js 18.17.1 o superior (recomendado 20 o 22).

```bash
npm install
npm run dev       # servidor de desarrollo en http://localhost:4321
npm run build     # genera el sitio en dist/
npm run preview   # sirve dist/ para revisarlo antes de publicar
```

Para trabajar contra una API local, cambia `API_URL` en `src/config.ts` (ver abajo) y arranca la API en
`http://localhost:3000`. La API debe tener el origen de la web en `CORS_ORIGINS`.

## Configuración: `src/config.ts`

Todo lo que cambia entre entornos está en un solo archivo:

| Constante | Para qué sirve |
|---|---|
| `API_URL` | Dirección de la API. Producción: `https://api.cashly.lat`. Desarrollo: `http://localhost:3000`. |
| `VERSION` | Versión que se muestra abajo a la derecha de la app y en el pie de la landing. Súbela en cada entrega. |
| `APP_NAME` | Nombre de la app (`Cashly`). Se usa en títulos, logo y textos. |
| `APP_DOWNLOAD_URL` | Enlace que abre el QR y el botón "Descargar la app" de la landing. **Pendiente:** cambiarlo por el enlace de App Store / TestFlight cuando la app esté publicada. |

> ⚠️ Antes de publicar, revisa que `API_URL` apunte a producción y no a `localhost`.

---

## Estructura del proyecto

```
src/
├── config.ts                  Configuración (API, versión, nombre, enlace de descarga)
├── assets/logo.jpeg           Logo; Astro lo optimiza y genera el favicon
├── styles/
│   ├── cashly-paleta.css      Paleta oficial de la marca (variables CSS)
│   └── global.css             Tailwind + paleta como clases + piezas reutilizables + tema claro
├── components/
│   ├── Logo.astro             Ícono + nombre, usado en landing y app
│   └── landing/               Secciones de la landing (una por archivo)
│       ├── Navbar.astro
│       ├── Hero.astro             Titular + vista previa del panel
│       ├── HowItWorks.astro       Los 3 pasos (Apple Pay → Atajo → panel)
│       ├── Features.astro         Funciones + ejemplo de meta
│       ├── Security.astro
│       ├── DownloadApp.astro      Bloque con el QR de descarga
│       ├── Faq.astro
│       └── Footer.astro
└── pages/
    ├── index.astro            Landing (solo junta las secciones)
    └── app.astro              Toda la aplicación: HTML + un <script> con la lógica
```

---

## Diseño y estilos

### La paleta

`src/styles/cashly-paleta.css` define los colores oficiales como variables CSS (`--cashly-azul`,
`--superficie`, `--texto-2`, `--cat-comida`…). **Si cambias un color ahí, cambia en toda la web.**

`global.css` convierte esas variables en clases de Tailwind:

| Clase | Color |
|---|---|
| `bg-fondo`, `bg-superficie`, `bg-superficie-2` | Fondos de página, tarjetas y campos |
| `text-texto`, `text-texto-2`, `text-texto-3` | Texto principal, secundario y de ayuda |
| `ring-borde`, `border-borde` | Bordes |
| `bg-brand`, `text-brand`, `bg-brand-cian` | Azul de marca y cian |
| `text-exito`, `text-alerta`, `text-error` | Estados |
| `bg-cat-comida`, `bg-cat-transporte`… | Color de cada categoría |
| `bg-cashly` / `text-cashly` | Degradado azul → cian (fondo o texto) |
| `bg-cashly-fondo` | Degradado azul oscuro del ícono de la app |

### Modo oscuro y modo claro

- La **landing** usa la paleta tal cual: modo oscuro.
- La **app** tiene la clase `tema-claro` en el `<body>`. Esa clase reasigna las variables a los colores
  "claros" de la paleta, así que las mismas clases (`bg-superficie`, `text-texto-2`…) se ven claras sin
  cambiar el HTML. Las piezas que deben seguir oscuras (panel de marca del login, tarjeta "Total gastado",
  cabecera del panel de admin) usan `bg-cashly-fondo` y colores blancos explícitos.

### Piezas reutilizables (`global.css`)

Para no repetir utilidades en el HTML de la app:

| Clase | Uso |
|---|---|
| `panel`, `panel-title` | Tarjeta blanca con borde y su título |
| `field`, `field-sm`, `field-code`, `field-label` | Campos de formulario, campo pequeño, campo de código de 6 dígitos y etiqueta |
| `btn-primary`, `btn-secondary`, `btn-danger`, `btn-link` | Botones |
| `kpi-label`, `kpi-value` | Etiqueta y número grande de los indicadores |
| `bar-track` | Fondo de las barras de progreso |
| `empty` | Mensaje de "no hay datos" |

---

## La landing (`/`)

Página estática, sin JavaScript. `index.astro` solo junta las secciones de `components/landing/`.
El texto de cada sección está en la parte superior del archivo correspondiente, en arreglos fáciles de
editar (pasos, funciones, preguntas frecuentes…).

**El QR de descarga** se genera al compilar en `DownloadApp.astro` a partir de `APP_DOWNLOAD_URL`. Lleva el
logo en el centro (con corrección de errores alta para que se siga leyendo). Para cambiar a dónde lleva,
solo cambia `APP_DOWNLOAD_URL` y vuelve a compilar.

---

## La aplicación (`/app`)

Toda la app vive en `src/pages/app.astro`: primero el HTML de cada pantalla y al final un `<script>` con la
lógica, dividido en secciones marcadas con comentarios `// ---------- Nombre ----------`.

### Acceso: registro, login y verificación

La pantalla de acceso tiene dos columnas en computador (panel de marca + formulario) y solo el formulario en
celular. Ninguna ruta de la API entrega la sesión directamente: siempre hay un segundo paso.

```
Crear cuenta ──► Llave de recuperación (se muestra una sola vez)
             ──► Código de 6 dígitos al correo (verify-email)
             ──► QR del Authenticator (primera vez) ──► Código del Authenticator ──► Sesión

Entrar ──────► Código del Authenticator (login/mfa) ──► Sesión
               (si el correo no está verificado, primero pide el código del correo)

¿Olvidaste la contraseña? ──► Correo + llave de recuperación + contraseña nueva (recover)
                          ──► Llave nueva + QR nuevo del Authenticator ──► Sesión
```

- El código del correo vence en 10 minutos y se puede reenviar cada 60 s (el botón muestra la cuenta atrás).
- El `mfaToken` del segundo paso vive solo en memoria; si vence, hay que volver a escribir la contraseña.

### Pestañas

| Pestaña | Qué muestra | Endpoints |
|---|---|---|
| 📊 **Resumen** | Total gastado, movimientos, tarjeta vs. efectivo, gasto por categoría, formulario de gasto manual y lista de transacciones (cambiar categoría o estado, borrar, filtrar) | `GET /v1/transactions`, `GET /v1/transactions/summary`, `POST/PATCH/DELETE /v1/transactions` |
| 📅 **Reportes** | Indicadores del período, gráfico diario de 30 días con promedio, reporte semanal con variación, categorías, top comercios y detalle por día | `GET /v1/reports/daily`, `GET /v1/reports/weekly` |
| 🎯 **Metas** | Límites mensuales (total o por categoría): cuánto queda, cuánto se puede gastar por día y proyección a fin de mes | `GET /v1/goals/progress`, `POST/PATCH/DELETE /v1/goals` |
| 👤 **Perfil** | Datos de la cuenta, cambiar contraseña con la llave de recuperación, tokens del Atajo de iPhone y cerrar sesión en todos los dispositivos | `POST /v1/auth/recover`, `GET/POST/DELETE /v1/ingest-tokens`, `POST /v1/auth/logout` |
| 🛡️ **Admin** | Solo para el administrador (ver abajo) | `GET /v1/admin/stats`, `GET/DELETE /v1/admin/users` |

**Datos de ejemplo:** si la API no tiene todavía los endpoints de Reportes o Metas (responde 404), la app
muestra datos de ejemplo con la misma forma y un aviso amarillo con el botón "Reintentar con la API real".

### Perfil y Atajo de iPhone

- **Cambiar contraseña** usa la llave de recuperación (`/v1/auth/recover`). La API entrega una llave nueva,
  reinicia el Authenticator y cierra todas las sesiones, así que la app vuelve a la pantalla de acceso para
  guardar la llave nueva y escanear el QR otra vez.
- **Tokens del Atajo:** cada iPhone usa un token propio para enviar los pagos a `POST /v1/ingest`. El token
  solo puede crear gastos. Se muestra **una sola vez** al crearlo y se puede revocar en cualquier momento.

### Panel de administración

- La pestaña 🛡️ Admin aparece **solo si la API dice que la cuenta es administradora**. Al entrar, la app
  consulta `GET /v1/admin/stats`: si responde, muestra la pestaña; con `403` (usuario normal) o `404` (API sin
  panel de admin) la deja oculta. La regla la decide la API: es la cuenta de `ADMIN_EMAIL` con el correo
  verificado y el Authenticator activo.
- El admin **solo ve metadatos**: la API no puede descifrar montos, comercios, categorías ni correos.
- Muestra: rango de fechas (7, 30, 90 días, 1 año o personalizado), usuarios, activos, transacciones, logins,
  gráfico de actividad por día (transacciones, registros, logins o metas), estado de las cuentas, uso y la
  tabla de cuentas con búsqueda por correo exacto.
- Puede **eliminar cuentas** (una en particular o la última creada). Siempre pide confirmación y la cuenta
  del administrador no se puede eliminar.

### Cómo habla con la API

Todas las peticiones pasan por la función `call(method, path, body, opciones)`:

- Agrega el token de sesión (`Authorization: Bearer …`) salvo que se pida `{ auth: false }`.
- Si la API responde `401`, renueva la sesión con el refresh token y repite la petición una vez. Si no se
  puede renovar, vuelve a la pantalla de acceso y avisa que la sesión venció.
- Convierte los errores de la API en mensajes legibles: errores de validación campo por campo, límite de
  intentos (`429`) con el tiempo de espera, y un mensaje claro si no se puede conectar (API caída o CORS).

### Qué se guarda en el navegador (`localStorage`)

| Clave | Contenido |
|---|---|
| `at` | Access token (sesión) |
| `rt` | Refresh token (para renovar la sesión) |
| `email` | Correo de la cuenta, solo para mostrarlo en Perfil (la API no lo guarda en claro) |
| `uid` | ID del usuario |

Al cerrar sesión se borran todas. Versiones anteriores guardaban también `apiUrl` e `it`; la app las limpia.

### Alertas y diálogos

- `toast(mensaje, tipo)` muestra una notificación arriba a la derecha. Tipos: `info`, `success`,
  `warning`, `error` (los errores duran más). Se cierra sola o con un clic.
- `ask({ title, message, ok, danger, input })` reemplaza a `confirm()` y `prompt()` del navegador con un
  diálogo propio. Devuelve `true`/`false`, o el texto escrito (`null` si se cancela) cuando se pasa `input`.

---

## Publicar

`npm run build` genera la carpeta `dist/` con archivos estáticos que se pueden subir a cualquier hosting
(S3 + CloudFront, Netlify, Vercel, etc.):

```
dist/
├── index.html        → /
├── app/index.html    → /app
└── _astro/           → CSS, JS e imágenes optimizadas
```

> Si el hosting es **S3 + CloudFront**, configura que `/app` sirva `/app/index.html` (por ejemplo, con una
> CloudFront Function o usando el endpoint de "website" de S3). Sin eso, `/app` puede dar error 403/404.

**Lista antes de publicar:**

1. `API_URL` apunta a `https://api.cashly.lat`.
2. Subiste `VERSION`.
3. La API tiene el dominio de la web en `CORS_ORIGINS`.
4. `npm run build` termina sin errores.
