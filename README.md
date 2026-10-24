# KODO — Tienda online

Proyecto completo de e-commerce para KODO (streetwear/oversize), listo para subir a un
hosting real. Incluye:

- Home, tienda por categorías (Remeras, Musculosas, Shorts, Pantalones) + Novedades
- Panel de producto lateral, carrito lateral funcional, checkout
- Pago online real con **Mercado Pago (Checkout Pro)**
- Envíos a todo Uruguay (el envío se paga al recibir, no está incluido en el precio)
- Seguimiento de pedido por número, con 4 estados
- Contacto (WhatsApp, Instagram, Email)
- **Panel de administración** (`/admin`) donde se puede editar TODO sin tocar código:
  textos, colores de marca, contacto, zonas/precios de envío, categorías, productos
  (con fotos, colores, talles, precio, descripción) y el estado de cada pedido.
- Diseño mobile-first, adaptado a tablet/escritorio, animaciones suaves.

## 1. Requisitos

- Node.js 18 o superior
- Una cuenta de Mercado Pago (Uruguay) para cobrar de verdad

## 2. Instalación local

```bash
cd kodo-store
npm install
cp .env.example .env
```

Editá `.env`:

```
ADMIN_USER=ariwolfson10
ADMIN_PASS=Ariel1061
SESSION_SECRET=poné-algo-largo-y-random-acá
MP_ACCESS_TOKEN=tu-access-token-de-mercado-pago
PUBLIC_URL=http://localhost:3000
```

Iniciar el servidor:

```bash
npm start
```

- Tienda: http://localhost:3000
- Admin: http://localhost:3000/admin (usuario y contraseña definidos en `.env`)

Sin `MP_ACCESS_TOKEN` configurado, el checkout igual funciona: crea el pedido y avisa
que el cobro online todavía no está activo, para que puedas probar todo el flujo antes
de conectar Mercado Pago.

## 3. Conseguir el Access Token de Mercado Pago

1. Entrá a https://www.mercadopago.com.uy/developers/panel/app con tu cuenta de KODO.
2. Creá una aplicación (tipo "Pagos online").
3. Copiá las **credenciales de producción** → `Access Token`.
4. Pegalo en `MP_ACCESS_TOKEN` en tu `.env` (o en las variables de entorno del hosting).
5. Nunca subas ese token a un repositorio público.

El servidor crea automáticamente una "preferencia de pago" por cada pedido y redirige
al comprador a Mercado Pago. Cuando el pago se aprueba, Mercado Pago le avisa al
servidor (webhook en `/api/mp/webhook`) y el pedido pasa a pagado + "PREPARANDO"
automáticamente.

**Importante:** el webhook necesita que `PUBLIC_URL` sea la URL real y pública de tu
sitio (no `localhost`) una vez que esté deployado, si no Mercado Pago no puede avisarle
al servidor que el pago se aprobó.

## 4. Dónde subir esto (hosting)

Cualquier hosting que corra Node.js sirve. Opciones simples y económicas:

- **Render.com** (fácil, tiene plan gratuito/económico) — "Web Service" desde este repo,
  build command `npm install`, start command `npm start`.
- **Railway.app** — similar a Render.
- Un VPS propio (DigitalOcean, etc.) con `pm2` o `systemd` para mantenerlo corriendo.

⚠️ **Muy importante sobre el almacenamiento**: este proyecto guarda los productos, los
textos/colores del sitio y los pedidos en archivos JSON dentro de `/data`, y las fotos
subidas en `/public/uploads`. Esto es simple y funciona perfecto en un VPS o en un plan
con **disco persistente**. Pero en hostings con sistema de archivos "efímero" (se borra
en cada redeploy), vas a perder esos datos. Si usás Render/Railway, activá un "disco
persistente" (Persistent Disk / Volume) apuntando a las carpetas `data/` y
`public/uploads/`. Si más adelante crece mucho el catálogo o el volumen de pedidos,
migrar `data/*.json` a una base de datos real (Postgres, MongoDB) es el siguiente paso
natural — el código está organizado en `server/db.js` justamente para que ese cambio
sea sencillo el día que haga falta.

Después de deployar, actualizá `PUBLIC_URL` en las variables de entorno del hosting con
la URL real (ej: `https://kodo.com.uy`).

## 5. Después de tener todo online

- Entrá a `/admin`, cargá los productos reales con sus fotos, precios, colores y talles.
- Reemplazá la foto del hero (Inicio → General y diseño → Foto de fondo del hero).
- Cargá el WhatsApp cuando lo tengan.
- Revisá las zonas y precios de envío.
- Hacé una compra de prueba de punta a punta con un monto bajo para confirmar que
  Mercado Pago está bien conectado antes de anunciar la tienda.

## 6. Seguridad — antes de anunciar la tienda

- Cambiá `SESSION_SECRET` por algo largo y random.
- Si querés, cambiá `ADMIN_PASS` a algo que no sea el que está en este ejemplo.
- Serví el sitio siempre con HTTPS (Render/Railway lo dan automático).
