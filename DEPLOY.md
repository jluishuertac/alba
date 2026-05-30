# ALBA SaaS — Guía de Despliegue en Cloudflare Workers

## Arquitectura seleccionada: Cloudflare Workers (Opción B)

Este proyecto usa TanStack Start con SSR + Nitro. El output es un **Cloudflare Worker** 
(no Pages estático). Cloudflare Pages daba 404 porque no había Worker que respondiera las 
solicitudes SSR — solo archivos estáticos sin un handler de fetch.

## Prerrequisitos

- Node.js 18+
- npm 9+
- Cuenta de Cloudflare con Workers habilitado
- Proyecto Supabase con las tablas migradas

## 1. Instalación

```bash
npm install
```

## 2. Variables de entorno — Desarrollo local

Crea un archivo `.env` en la raíz:

```env
VITE_SUPABASE_URL=https://xxxxxxxxxxxxxxxxxxxx.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_URL=https://xxxxxxxxxxxxxxxxxxxx.supabase.co
SUPABASE_PUBLISHABLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## 3. Build de producción

```bash
npm run build
```

Esto produce:
- `dist/client/`  — assets estáticos (JS, CSS, imágenes)
- `dist/server/_worker.js` — el Cloudflare Worker

## 4. Preview local del Worker

```bash
npm run preview:worker
```

Abre http://localhost:8788

## 5. Variables de entorno — Producción (Cloudflare)

Ejecuta estos comandos **antes del primer deploy**:

```bash
wrangler secret put VITE_SUPABASE_URL
wrangler secret put VITE_SUPABASE_PUBLISHABLE_KEY
wrangler secret put SUPABASE_URL
wrangler secret put SUPABASE_PUBLISHABLE_KEY
wrangler secret put SUPABASE_SERVICE_ROLE_KEY
```

Cada comando pedirá el valor de forma interactiva (no aparece en el historial de terminal).

## 6. Deploy

```bash
# Build + deploy en un solo comando:
npm run deploy

# O paso a paso:
npm run build
wrangler deploy
```

## 7. Verificación post-deploy

Comprueba en la terminal que wrangler muestre:

```
✨  Built successfully
Total Upload: ~XXX KiB / gzip: ~XX KiB
Uploaded alba-saas (X sec)
Published alba-saas (X sec)
  https://alba-saas.<tu-subdominio>.workers.dev
```

Luego visita la URL y verifica:

- [ ] La landing `/` carga correctamente
- [ ] `/auth?mode=signup` muestra el formulario de registro
- [ ] `/auth?mode=login` muestra el formulario de login
- [ ] Login con email/password funciona
- [ ] Login con Google funciona (si está configurado en Supabase)
- [ ] `/dashboard` redirige a login si no estás autenticado
- [ ] `/dashboard` carga correctamente si estás autenticado
- [ ] No hay errores 404 en ninguna ruta
- [ ] Los assets (CSS, JS) cargan con status 200
- [ ] El favicon aparece correctamente

## Dominio personalizado

En el panel de Cloudflare → Workers → tu worker → Triggers → Custom Domains:
Agrega tu dominio (ej: `app.plato.com`).

Recuerda actualizar en Supabase → Authentication → URL Configuration:
- Site URL: `https://app.plato.com`
- Redirect URLs: `https://app.plato.com/**`

## Migraciones de Supabase

Para aplicar las migraciones al proyecto de Supabase en producción:

```bash
# Instala Supabase CLI si no lo tienes
npm install -g supabase

# Enlaza con tu proyecto
supabase link --project-ref <project-id>

# Aplica migraciones
supabase db push
```

El `<project-id>` está en `supabase/config.toml` como `project_id`.
