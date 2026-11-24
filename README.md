# nube-bank_2
Es un banco de nube en el cual puedes consultar tu saldo 
# NubeBanc

Proyecto final del Diplomado Cloud Computing + AWS Cloud Practitioner. Es la página web de un banco ficticio con arquitectura de nube híbrida. No es una entidad financiera real: no captures datos reales en ella.

## Qué incluye

```
banco-nube/
├── frontend/index.html        Sitio (HTML, CSS y JavaScript). Nube pública.
├── backend/server.js          API REST en Node.js, sin dependencias. Nube pública.
├── private-ledger/ledger.py   Libro mayor en Python, sin dependencias. Nube privada.
└── README.md
```

El sitio tiene inicio, servicios, simulador de crédito, acceso demo con panel de cuentas y movimientos, y una sección que muestra en vivo si la API y el libro mayor están conectados.

## Cómo funciona la nube híbrida

```
Navegador ──HTTPS──> Sitio estático (nube pública)
    │
    └──HTTPS──> API Node.js (nube pública) ──HTTP + token──> Libro mayor Python (nube privada)
```

- La parte que ve el cliente y la API van en la nube pública.
- Los saldos y movimientos viven en la nube privada. El libro mayor solo escucha en `127.0.0.1` y exige un token.
- Si el libro mayor no responde, la API usa datos de respaldo y lo avisa (`origen: "respaldo"`). El sitio sigue funcionando y también tiene datos locales por si falla la API.

## Correrlo en tu computadora

Necesitas Node.js 18 o superior y Python 3.

```bash
# Terminal 1: libro mayor (nube privada)
python3 private-ledger/ledger.py

# Terminal 2: API (nube pública)
node backend/server.js

# Terminal 3: sitio
cd frontend && python3 -m http.server 8080
```

Abre http://localhost:8080 y entra con usuario `demo`, contraseña `demo123`.

## Variables de entorno

| Variable | Dónde | Para qué | Por defecto |
|---|---|---|---|
| `PORT` | API | Puerto de la API | `3000` |
| `LEDGER_URL` | API | Dirección del libro mayor | `http://127.0.0.1:5001` |
| `LEDGER_TOKEN` | API y libro mayor | Token compartido entre los dos | vacío (sin token) |
| `ALLOWED_ORIGIN` | API | Origen permitido por CORS | `*` |
| `LEDGER_HOST`, `LEDGER_PORT` | Libro mayor | Dónde escucha | `127.0.0.1`, `5001` |

En producción pon `ALLOWED_ORIGIN` con la dirección de tu sitio y define un `LEDGER_TOKEN` largo y aleatorio en la API y en el libro mayor.

## Desplegarlo con servicios gratuitos

Revisa los límites y precios vigentes de cada servicio antes de usarlos: cambian.

1. **Sitio:** sube la carpeta `frontend/` a un hosting estático gratuito (GitHub Pages, Netlify o Cloudflare Pages). Con AWS sería S3 + CloudFront, que se ve en la sesión 3.
2. **API:** despliega `backend/` en un servicio PaaS con plan gratuito (por ejemplo Render), como en la sesión 7. Define `LEDGER_URL`, `LEDGER_TOKEN` y `ALLOWED_ORIGIN`.
3. **Libro mayor (nube privada):** córrelo en tu computadora o en una Raspberry Pi. Para que la API en la nube lo alcance, expónlo con un túnel (por ejemplo Cloudflare Tunnel) y mantén el token activo. Si no puedes exponerlo, la API trabaja con el respaldo y lo dices en tu presentación.
4. **Dominio:** las plataformas dan un subdominio gratis. Un dominio propio normalmente cuesta; con AWS se gestiona en Route 53 (sesión 3).
5. En el sitio, pega la URL pública de la API en el campo "URL de la API" de la sección Arquitectura, o define `window.NUBEBANC_API` antes del script.

## Mapa a las 10 sesiones

| Sesión | Dónde se ve en el proyecto |
|---|---|
| 1. Introducción a la nube | Nube pública, privada e híbrida; costos del plan gratuito |
| 2. Desarrollo web 1 | `index.html`, HTTP/HTTPS, puertos 3000, 5001 y 8080, CORS |
| 3. Desarrollo web 2 | CSS y JavaScript del sitio; hosting estático y dominio (S3, CloudFront, Route 53) |
| 4. APIs y servicios REST | Rutas `/api/health`, `/api/login`, `/api/accounts`, `/api/transactions`, `/api/loan/simulate` |
| 5. APIs con backend sencillo | `backend/server.js` en Node.js |
| 6. Consumo de APIs desde el navegador | `fetch` en el sitio, con tiempo límite y respaldo |
| 7. Despliegue con PaaS | Despliegue de la API |
| 8. Integración front-end y back-end | URL de la API configurable, token de sesión, token entre API y libro mayor |
| 9. Virtualización y contenedores | Siguiente paso: un `Dockerfile` para la API y el libro mayor |
| 10. Proyecto final | Seguridad, fiabilidad (respaldo) y costo bajo, ideas del marco Well-Architected |

## Límites de esta versión

- Hay un solo usuario de demostración y las sesiones viven en memoria: se pierden si reinicias la API.
- Los datos no se guardan en una base de datos. Un siguiente paso natural es DynamoDB o RDS (sesión 6).
- Es una demostración: no la uses con datos ni dinero reales.
