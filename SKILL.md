# security-audit

## Propósito

Auditoría de seguridad sistemática del sitio generado antes del deploy. Cubre headers HTTP, secrets hardcodeados, configuración de `.gitignore`, dependencias vulnerables, links externos inseguros, exposición de datos sensibles en el bundle de cliente, y los vectores del OWASP Top 10 aplicables a sitios estáticos con Next.js. Se ejecuta automáticamente en la Fase 5 (QA), antes de preguntar al usuario si quiere hacer deploy.

---

## Cuándo activarse

- Automáticamente al final de la Fase 5, antes de reportar el resultado al usuario.
- Cuando el usuario dice "revisar seguridad", "auditoría", "¿es seguro?", o "preparar para producción".
- Antes de cualquier deploy a Vercel o plataforma externa.
- Cuando se agregaron integraciones nuevas (pagos, formularios, terceros).

---

## Instrucciones

### 1. HEADERS DE SEGURIDAD EN `next.config.js`

Verificar que el archivo `next.config.js` del proyecto generado incluye `async headers()`. Si no existe o está vacío, agregar la configuración completa:

```js
// next.config.js — configuración mínima de seguridad
const securityHeaders = [
  {
    key: 'X-Frame-Options',
    value: 'DENY',
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff',
  },
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin',
  },
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=()',
  },
  {
    // 'unsafe-inline' en script-src es necesario para los scripts de hidratación de Next.js.
    // 'unsafe-inline' en style-src es necesario para las clases de Tailwind CSS.
    key: 'Content-Security-Policy',
    value: [
      "default-src 'self'",
      "script-src 'self' 'unsafe-inline' 'unsafe-eval'",
      "style-src 'self' 'unsafe-inline'",
      "img-src 'self' data: blob: https:",
      "font-src 'self' data:",
      "connect-src 'self' https://formspree.io https://wa.me https://api.whatsapp.com",
      "frame-ancestors 'none'",
      "object-src 'none'",
      "base-uri 'self'",
    ].join('; '),
  },
]

const nextConfig = {
  outputFileTracingRoot: require('path').join(__dirname),
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: securityHeaders,
      },
    ]
  },
}

module.exports = nextConfig
```

**Ajustar `connect-src`** según las integraciones del proyecto:
- Formspree: `https://formspree.io`
- Cal.com: `https://cal.com https://*.cal.com`
- Mercado Pago: `https://api.mercadopago.com`
- Stripe: `https://api.stripe.com https://js.stripe.com`
- PayPal: `https://api-m.paypal.com https://api-m.sandbox.paypal.com`
- Google Analytics: `https://www.google-analytics.com https://www.googletagmanager.com`

**Por qué cada header:**
- `X-Frame-Options: DENY` → impide que el sitio sea embebido en un `<iframe>` (previene clickjacking)
- `X-Content-Type-Options: nosniff` → el browser no intenta adivinar el Content-Type de las respuestas
- `Referrer-Policy: strict-origin-when-cross-origin` → solo envía el origen (no la URL completa) en requests cross-origin
- `Permissions-Policy` → deshabilita acceso a cámara, micrófono y geolocalización que el sitio no usa
- `Content-Security-Policy` → lista blanca de orígenes autorizados para scripts, estilos, imágenes y conexiones

### 2. CHECKLIST DE SECRETS HARDCODEADOS (60+ patrones)

Buscar en todos los archivos `.ts`, `.tsx`, `.js`, `.json` del proyecto generado (excluyendo `node_modules` y `.next`):

```bash
# Ejecutar en la carpeta del proyecto generado
grep -rn \
  "sk_live_\|sk_test_\|pk_live_\|pk_test_\|rk_live_\|rk_test_\|\
  AAAA[A-Za-z0-9_-]{50,}\|\
  AIza[0-9A-Za-z-_]{35}\|\
  ya29\.[0-9A-Za-z\-_]+\|\
  [Aa]ccess[_-]?[Tt]oken\|[Aa]ccess[_-]?[Kk]ey\|\
  [Aa][Pp][Ii][_-]?[Kk]ey\|\
  [Ss]ecret[_-]?[Kk]ey\|[Ss]ecret[_-]?[Tt]oken\|\
  [Pp]rivate[_-]?[Kk]ey\|[Cc]lient[_-]?[Ss]ecret\|\
  [Aa]uth[_-]?[Tt]oken\|[Bb]earer [A-Za-z0-9_\-\.=]+\|\
  ghp_[A-Za-z0-9]{36}\|gho_[A-Za-z0-9]{36}\|\
  ghs_[A-Za-z0-9]{36}\|github_pat_[A-Za-z0-9_]{82}\|\
  xox[baprs]-[0-9]{10,}-[A-Za-z0-9]{24,}\|\
  T[0-9A-Z]{8}\.AE[0-9A-Za-z]{24}\|\
  [Aa][Ww][Ss][_-][Ss]ecret\|ASIA[A-Z0-9]{16}\|AKIA[A-Z0-9]{16}\|\
  [Pp]assword\s*[:=]\s*['\"][^'\"]{6,}\|\
  [Pp]asswd\s*[:=]\s*['\"][^'\"]{6,}\|\
  [Mm]ongo\|[Mm]y[Ss][Qq][Ll]\|[Pp]ostgres\|[Ss][Qq][Ll]ite\|\
  mongodb\+srv://\|postgres://\|mysql://\|\
  -----BEGIN [A-Z]+ PRIVATE KEY-----\|\
  [Ss][Mm][Tt][Pp].*[Pp]assword\|smtp://.*:.*@\|\
  twilio.*[Ss][Ii][Dd]\|AC[a-zA-Z0-9]{32}\|\
  sendgrid\|SG\.[A-Za-z0-9_-]{22}\.[A-Za-z0-9_-]{43}\|\
  mailgun\|key-[0-9a-zA-Z]{32}\|\
  APP_SECRET\|JWT_SECRET\|SESSION_SECRET\|ENCRYPTION_KEY\|\
  firebase.*[Aa][Pp][Ii][Kk]ey\|firebaseConfig\|\
  supabase.*[Ss]ervice[Rr]ole\|eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9" \
  --include="*.ts" --include="*.tsx" --include="*.js" --include="*.json" \
  . 2>/dev/null | grep -v node_modules | grep -v .next | grep -v package-lock
```

**Si hay resultados → bloquear el build y reportar al usuario antes de proceder.**

**Excepciones aceptables (no son secrets reales):**
- Placeholders como `YOUR_API_KEY`, `XXXXXXXXXXXX`, `YOUR_FORMSPREE_ID`
- Valores de ejemplo en comentarios
- Variables de entorno referenciadas como `process.env.ALGO` (sin el valor hardcodeado)
- Client IDs de PayPal/Stripe marcados como `NEXT_PUBLIC_` (son seguros para el cliente)

### 3. VERIFICAR `.gitignore`

El `.gitignore` del proyecto generado debe incluir al menos estas líneas:

```gitignore
# Variables de entorno — NUNCA commitear
.env
.env.local
.env.*.local
.env.development.local
.env.test.local
.env.production.local
*.pem
```

Si alguna de estas líneas falta, agregarla antes del commit inicial del proyecto.

Verificación:
```bash
grep -E "\.env|\.pem" .gitignore || echo "FALTA: agregar protección de .env en .gitignore"
```

### 4. `npm audit` — INTERPRETACIÓN Y ACCIÓN

Ejecutar dentro de la carpeta del proyecto:
```bash
npm audit 2>&1
```

**Cómo interpretar resultados:**

| Nivel | Acción |
|---|---|
| `critical` | Fix inmediato antes del deploy. Correr `npm audit fix` (sin `--force` primero). Si requiere `--force`, evaluar manualmente. |
| `high` | Fix antes del deploy si el vector de ataque es alcanzable desde el browser. Si solo afecta CLI/build time, documentar y continuar. |
| `moderate` | Fix deseable. Evaluar si afecta producción o solo desarrollo. |
| `low` | Aceptable para deployar. Documentar y hacer seguimiento. |

**Cuándo NO usar `npm audit fix --force`:**
- Cuando actualiza a una versión major que puede romper APIs (ej: Next.js 14 → 15 requiere verificar breaking changes)
- Cuando la fix instala versiones incompatibles con otras dependencias

**Proceso correcto:**
```bash
# 1. Intentar fix automático sin breaking changes
npm audit fix

# 2. Verificar build después del fix
npm run build

# 3. Solo si el build pasa, proceder
# 4. Si hay vulnerabilidades críticas que requieren --force, evaluar manualmente
```

### 5. LINKS EXTERNOS — HTTPS Y ATRIBUTOS DE SEGURIDAD

Buscar todos los `href` que no sean internos (`#` o `/ruta`):

```bash
grep -rn "href=\"http://" --include="*.tsx" --include="*.ts" . | grep -v node_modules
```

**Todo link a URL externa debe tener:**
```tsx
// CORRECTO
<a
  href="https://dominio.com"
  target="_blank"
  rel="noopener noreferrer"
  aria-label="Descripción del destino (abre en nueva pestaña)"
>

// MAL — sin rel abre un riesgo de window.opener hijacking
<a href="https://dominio.com" target="_blank">
```

**Verificar:**
- Todos los `target="_blank"` tienen `rel="noopener noreferrer"` ✓/✗
- No hay `http://` sin S en links de producción ✓/✗
- Los links de WhatsApp usan `https://wa.me/` no `http://` ✓/✗

### 6. DATOS SENSIBLES EN EL BUNDLE DE CLIENTE

Todo lo que esté en un componente marcado con `'use client'` o en un Server Component que pase props al cliente **es visible en el bundle descargado por el browser**.

**Nunca exponer en el cliente:**
- API keys secretas (sin `NEXT_PUBLIC_` prefix)
- Access tokens de servicios (Stripe secret, MP access token)
- Contraseñas de base de datos
- Tokens JWT de sesión
- Claves privadas de cualquier tipo

**Verificación:**
```bash
# Buscar variables de entorno sin NEXT_PUBLIC_ en componentes cliente
grep -rn "process\.env\." --include="*.tsx" . | grep -v "NEXT_PUBLIC" | grep -v node_modules
```

Cada resultado debe revisarse manualmente. Si `process.env.ALGO_SIN_PUBLIC` aparece en un Client Component (`'use client'`), es una fuga — moverlo a un Server Component o a una API Route.

**Regla general para this project:**
- `NEXT_PUBLIC_PAYPAL_CLIENT_ID` → seguro en cliente (es público)
- `PAYPAL_SECRET` → solo en API routes del servidor
- `MP_ACCESS_TOKEN` → solo en API routes del servidor
- `STRIPE_SECRET_KEY` → solo en API routes del servidor

### 7. OWASP TOP 10 — APLICABLE A NEXT.JS ESTÁTICO

Los sitios generados con MyCloudeWEB son principalmente estáticos. El vector de ataque es reducido, pero estos riesgos aplican:

**A01 — Broken Access Control**
- Aplica si: el proyecto tiene API routes (`app/api/`)
- Verificar: cada API route valida que el request viene de un origen autorizado
- Verificar: no hay rutas de admin accesibles sin autenticación

**A02 — Cryptographic Failures**
- Aplica: HTTPS obligatorio en producción (Vercel lo maneja automáticamente)
- Verificar: no hay secrets transmitidos en query params (URLs)
- Verificar: formularios usan `https://` para el endpoint de envío

**A03 — Injection**
- Aplica si: hay campos de input que se procesan en el servidor
- Para formularios que van a Formspree: Formspree maneja la sanitización
- Para API routes propias: sanitizar siempre el input con `DOMPurify` o equivalente

**A05 — Security Misconfiguration**
- Verificar: `next.config.js` tiene headers de seguridad (ver sección 1)
- Verificar: variables de entorno no están en el bundle del cliente
- Verificar: el modo debug/dev no está habilitado en producción

**A06 — Vulnerable and Outdated Components**
- Verificar: `npm audit` no tiene vulnerabilidades críticas o altas explotables
- Verificar: Next.js está en la última versión estable

**A07 — Identification and Authentication Failures**
- Solo aplica si el proyecto tiene login/auth
- Para sitios sin auth (mayoría de los generados por MyCloudeWEB): no aplica

**A08 — Software and Data Integrity Failures**
- Verificar: scripts de terceros se cargan con `next/script` (no desde CDNs arbitrarios sin integrity hash)
- Verificar: dependencias vienen de npm, no de URLs directas

**A09 — Security Logging and Monitoring Failures**
- Para sitios estáticos: instalar `@vercel/analytics` como mínimo para detectar comportamiento anómalo

**A10 — Server-Side Request Forgery (SSRF)**
- Aplica si: hay API routes que hacen fetch a URLs externas basadas en input del usuario
- Para Mercado Pago/PayPal/Stripe: las URLs son fijas en el código — no hay riesgo

### 8. CHECKLIST FINAL PRE-DEPLOY

Ejecutar en orden antes de reportar al usuario:

```
HEADERS
- [ ] next.config.js tiene async headers() configurado
- [ ] X-Frame-Options presente
- [ ] X-Content-Type-Options presente
- [ ] Referrer-Policy presente
- [ ] Permissions-Policy presente
- [ ] Content-Security-Policy presente

SECRETS
- [ ] 0 resultados en búsqueda de 60+ patrones de secrets
- [ ] .gitignore incluye .env*.local y *.pem
- [ ] Variables sensibles usan process.env (no valores directos)

DEPENDENCIAS
- [ ] npm audit: 0 vulnerabilidades críticas o altas explotables en producción

LINKS
- [ ] 0 links con http:// (sin S)
- [ ] Todos los target="_blank" tienen rel="noopener noreferrer"

CLIENTE
- [ ] 0 API keys secretas en Client Components
- [ ] Solo NEXT_PUBLIC_ variables en el bundle del cliente
```

**Si algún ítem crítico falla → corregir antes de preguntar por el deploy.**
**Si todos pasan → reportar al usuario con el resultado y preguntar si quiere hacer deploy.**

### Reporte al usuario

Al finalizar la auditoría, presentar un resumen breve:

```
## Auditoría de seguridad completada

✓ Headers de seguridad configurados
✓ Sin secrets hardcodeados
✓ .gitignore protege variables de entorno
✓ Links externos con HTTPS y rel correcto
✓ npm audit: [X] vulnerabilidades — [ninguna crítica / ver detalle]

El sitio está listo para deploy.
```

Si hay problemas:
```
## Auditoría de seguridad — acción requerida

✗ [descripción del problema]
→ [corrección aplicada / corrección necesaria]

[continúa después de la corrección]
```

---

## Ejemplos

### Resultado de búsqueda de secrets — sin problemas

```bash
$ grep -rn "sk_live_\|api_key\|SECRET" --include="*.tsx" . | grep -v node_modules
./lib/config.ts:3:export const PHONE_NUMBER = 'XXXXXXXXXXX'
# → ACEPTABLE: es un placeholder explícito, no un valor real
```

### Resultado de búsqueda de secrets — problema encontrado

```bash
$ grep -rn "sk_live_" --include="*.tsx" . | grep -v node_modules
./components/Pricing.tsx:45:const stripeKey = 'sk_live_abc123...'
# → BLOQUEANTE: API key real hardcodeada en un componente cliente
# → Acción: mover a process.env.STRIPE_SECRET_KEY en una API route
```

### CSP ajustada para proyecto con Cal.com + Stripe

```js
"connect-src 'self' https://formspree.io https://wa.me https://api.whatsapp.com https://cal.com https://*.cal.com https://api.stripe.com https://js.stripe.com",
```
