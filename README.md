# Verona Store — Frontend + Backend de pedidos con IA

Este proyecto tiene dos partes en el mismo repositorio:

- `index.html`: la app web (HTML + Tailwind + JavaScript). Funciona sola y guarda los pedidos en `localStorage`.
- `server.js`: el backend en Node.js/Express que interpreta mensajes de WhatsApp/Messenger con Gemini y devuelve el pedido en JSON.

> GitHub solo guarda el código. Para que el backend esté en línea se despliega desde GitHub a un servicio como **Render**. GitHub Pages solo puede publicar el `index.html` (es estático).

---

## 1. Estructura

```text
.
├── index.html          # Frontend
├── server.js           # Backend (Express + Gemini)
├── package.json
├── render.yaml         # Configuración opcional para Render
├── .node-version       # Fuerza Node 22 en Render
├── .env.example        # Variables de entorno de ejemplo
├── .gitignore
└── README.md
```

---

## 2. Requisitos

- Node.js 22 o superior (`node --version`).
- Una API key de Google AI Studio: https://aistudio.google.com/apikey
- Una cuenta de GitHub.
- Una cuenta de Render (o Railway/Fly.io).

---

## 3. Probar el backend en local

Desde la carpeta del proyecto:

```powershell
npm install
Copy-Item .env.example .env
notepad .env
```

En `.env` coloca tu clave real:

```env
GEMINI_API_KEY=AIza...tu_clave...
GEMINI_MODEL=gemini-2.5-flash
ALLOWED_ORIGINS=*
```

Arranca el servidor:

```powershell
npm start
```

Debe aparecer:

```text
[verona-backend] Escuchando en el puerto 3000 con el modelo gemini-2.5-flash
```

Prueba el endpoint con PowerShell:

```powershell
$body = @{
  messageText = 'Hola, 2 pastes de nutella y 1 hawaiano para Ana'
  catalogProducts = @(
    @{ id = '8';  name = 'Paste de Nutella'; category = 'Pastes' },
    @{ id = '10'; name = 'Paste Hawaino';    category = 'Pastes' }
  )
  availableCategories = @('Pastes')
} | ConvertTo-Json -Depth 5

Invoke-RestMethod `
  -Uri http://localhost:3000/api/parse-order `
  -Method Post `
  -ContentType 'application/json' `
  -Body $body
```

Respuesta esperada (el orden puede variar):

```json
{
  "customerName": "Ana",
  "items": [
    { "productId": "8",  "productName": "Paste de Nutella", "category": "Pastes", "qty": 2 },
    { "productId": "10", "productName": "Paste Hawaino",    "category": "Pastes", "qty": 1 }
  ],
  "warnings": []
}
```

También puedes abrir `http://localhost:3000/health` en el navegador.

---

## 4. Endpoint

### `POST /api/parse-order`

**Body:**

```json
{
  "messageText": "Hola, 2 de nutella y 1 hawaiano para Ana",
  "catalogProducts": [
    { "id": "8", "name": "Paste de Nutella", "category": "Pastes" }
  ],
  "availableCategories": ["Pastes", "Pizzas", "Pan de muerto"]
}
```

**Respuesta 200:**

```json
{
  "customerName": "Ana",
  "items": [
    {
      "productId": "8",
      "productName": "Paste de Nutella",
      "category": "Pastes",
      "qty": 2
    }
  ],
  "warnings": []
}
```

**Errores:**

| Código | Significado |
|---|---|
| 400 | Falta el mensaje o el catálogo está vacío |
| 413 | Mensaje o cuerpo demasiado grande |
| 429 | Demasiadas solicitudes por minuto |
| 502 | Gemini devolvió algo inválido o falló |
| 504 | Gemini tardó más de 30 segundos |

El backend valida que los productos devueltos existan en el catálogo y que las cantidades sean enteros entre 1 y 99. Si Gemini inventa un producto, se descarta y se agrega una advertencia.

---

## 5. Crear el repositorio en GitHub

1. Entra a https://github.com/new
2. Nombre sugerido: `verona-store`
3. Puede ser **privado** o público.
4. **No** marques “Add a README” ni “Add .gitignore” (ya los tienes).
5. Crea el repositorio.

Después, en PowerShell, desde la carpeta del proyecto:

```powershell
cd "C:\Users\Rodrigo\Documents\Verona Store\Nueva carpeta"

git init
git add .
git commit -m "Verona: frontend y backend de pedidos con IA"

git branch -M main
git remote add origin https://github.com/TU-USUARIO/verona-store.git
git push -u origin main
```

Si Git te pide iniciar sesión, usa tu cuenta de GitHub o un token personal.

Comprueba que `.env` **no** se subió:

```powershell
git status
```

Debe aparecer limpio y `.env` debe estar ignorado por `.gitignore`.

---

## 6. Desplegar el backend en Render

1. Crea una cuenta en https://render.com y conéctala con GitHub.
2. Dashboard → **New +** → **Web Service**.
3. Selecciona el repositorio `verona-store`.
4. Configura:

| Campo | Valor |
|---|---|
| Name | `verona-backend` |
| Runtime | `Node` |
| Root Directory | *(vacío)* |
| Build Command | `npm install` |
| Start Command | `npm start` |
| Health Check Path | `/health` |

5. En **Environment Variables** agrega:

| Variable | Valor |
|---|---|
| `GEMINI_API_KEY` | Tu clave de Google AI Studio |
| `GEMINI_MODEL` | `gemini-2.5-flash` |
| `ALLOWED_ORIGINS` | `*` mientras pruebas; luego la URL de tu frontend |

6. Pulsa **Create Web Service** y espera el deploy.
7. Copia la URL que te da Render, por ejemplo:

```text
https://verona-backend.onrender.com
```

También puedes usar el archivo `render.yaml`: Render lo detecta y crea el servicio, pero te pedirá la `GEMINI_API_KEY`.

> **Free tier:** el servicio se “duerme” tras un rato sin uso. La primera petición puede tardar 30-60 segundos. Si eso es un problema, usa un plan pago o un servicio que no duerma.

---

## 7. Publicar el frontend en GitHub Pages (opcional)

1. En GitHub, entra al repositorio → **Settings** → **Pages**.
2. Source: **Deploy from a branch**.
3. Branch: `main`, carpeta `/ (root)`.
4. Guarda. GitHub te dará una URL como:

```text
https://TU-USUARIO.github.io/verona-store/
```

El frontend no necesita build: es un `index.html` con CDN de Tailwind y FontAwesome.

Si prefieres no usar GitHub Pages, puedes abrir `index.html` localmente o subirlo a Netlify/Vercel.

---

## 8. Conectar el frontend con el backend

En `index.html`, busca:

```js
const BACKEND_URL = localStorage.getItem('verona_backend_url')
    || ((location.hostname === 'localhost' || location.hostname === '127.0.0.1')
        ? 'http://localhost:3000'
        : 'https://veronastore-bkend.onrender.com');
```

Cambia la URL de producción por la que te dio Render, o hazlo sin editar el archivo desde la consola del navegador:

```js
localStorage.setItem('verona_backend_url', 'https://TU-SERVICIO.onrender.com');
location.reload();
```

**Importante:** si publicas el frontend en GitHub Pages, configura `ALLOWED_ORIGINS` en Render con el origen exacto, sin barra final:

```env
ALLOWED_ORIGINS=https://TU-USUARIO.github.io
```

Si usas `*`, cualquier sitio puede llamar a tu backend y consumir tu cuota de Gemini.

---

## 9. Seguridad y costos

- La `GEMINI_API_KEY` vive **solo** en el backend; nunca la pongas en `index.html`.
- No subas `.env` a GitHub.
- El endpoint tiene rate limit por IP (30 peticiones/minuto por defecto).
- Revisa el precio y la cuota gratuita de Gemini en Google AI Studio.
- Si el negocio crece, mueve el rate limit a Redis y agrega autenticación por usuario.

---

## 10. Problemas comunes

| Síntoma | Causa probable | Solución |
|---|---|---|
| `Falta GEMINI_API_KEY` | No hay `.env` o Render no tiene la variable | Configúrala |
| `404 model not found` | El modelo no está disponible para tu clave | Cambia `GEMINI_MODEL` (por ejemplo `gemini-3.8-flash`) |
| `429` | Rate limit de Render o de Gemini | Espera o sube el límite |
| `502` | Gemini devolvió JSON inválido o falló | Revisa los logs de Render |
| `504` | Gemini tardó más de 30 s | Reintenta o sube `REQUEST_TIMEOUT_MS` |
| Error de CORS en el navegador | `ALLOWED_ORIGINS` no coincide | Pon la URL exacta del frontend |
| El botón queda en “Analizando...” | Cold start de Render | Espera; el frontend aborta a los 45 s |

---

## 11. Flujo de la IA

1. El usuario pega el mensaje en el panel “Intérprete de Pedidos IA”.
2. El frontend envía el texto + el catálogo actual (ids, nombres, categorías).
3. El backend construye un prompt con el catálogo y pide a Gemini JSON estructurado.
4. El backend valida ids, cantidades y catálogo; descarta lo que no existe.
5. El frontend muestra una vista previa con los productos detectados.
6. El usuario decide si suma al pedido actual o lo reemplaza, y aplica.
