# WebGuard

Auditoría de seguridad automática para sitios web Next.js, integrada directamente en Claude Code.

WebGuard analiza tu sitio antes del deploy y detecta vulnerabilidades reales: secrets expuestos en el código, headers HTTP mal configurados, dependencias con fallas conocidas, formularios inseguros y vectores del OWASP Top 10. Al terminar, te entrega un reporte claro con las correcciones necesarias.

---

## Instalación

```bash
git clone https://github.com/luisrecalde/webguard ~/.claude/skills/webguard
```

Un solo comando. Sin configuración adicional.

---

## Cómo usarlo

Una vez instalado, tenés dos formas de activarlo:

**Con el comando directo:**
```
/webguard
```

**Con lenguaje natural** — simplemente decile a Claude:
- *"Auditá la seguridad de mi sitio"*
- *"¿El sitio está listo para publicar?"*
- *"Revisá la seguridad antes del deploy"*
- *"¿Es seguro?"*

WebGuard también se activa automáticamente cuando agregás integraciones nuevas (pagos, formularios, terceros) o antes de cualquier deploy a producción.

---

## Qué analiza

### Headers HTTP de seguridad
Verifica que `next.config.js` tenga configurados los headers que los browsers modernos exigen. Si faltan, WebGuard los agrega con la configuración correcta: `X-Frame-Options`, `Content-Security-Policy`, `X-Content-Type-Options`, `Referrer-Policy` y `Permissions-Policy`.

### Secrets hardcodeados
Escanea el proyecto con más de 60 patrones para detectar API keys, tokens de acceso, contraseñas, claves privadas y credenciales de bases de datos expuestas en el código fuente. Si encuentra algo, bloquea el deploy.

### Dependencias vulnerables
Ejecuta `npm audit` e interpreta los resultados con criterio: distingue entre vulnerabilidades que tienen riesgo real en producción y las que solo afectan al entorno de desarrollo.

### Configuración de `.gitignore`
Verifica que los archivos de variables de entorno (`.env`, `.env.local`, `*.pem`) estén correctamente excluidos del repositorio. Si faltan las entradas, las agrega.

### Links externos
Detecta links sin HTTPS y atributos de seguridad faltantes (`rel="noopener noreferrer"`) que pueden exponer a tus visitantes a ataques de redirección.

### Datos sensibles en el bundle del cliente
Revisa que ninguna API key secreta ni token privado quede expuesto en el código JavaScript que descarga el browser del visitante.

### Formularios y envíos
Verifica que los endpoints de formularios usen HTTPS y que los datos de contacto viajen de forma segura.

### OWASP Top 10
Cubre los vectores de ataque relevantes para sitios Next.js: Broken Access Control, Cryptographic Failures, Injection, Security Misconfiguration, Vulnerable Components, Software Integrity Failures y SSRF.

---

## Para quién es

WebGuard está pensado para dueños de sitios web y desarrolladores que quieren publicar con la certeza de que su sitio es seguro, sin necesidad de ser expertos en ciberseguridad.

El análisis es automático. El reporte es directo. Las correcciones son concretas.

Si algo está mal, WebGuard lo dice con claridad y lo corrige antes de que el sitio salga al aire.

---

## Licencia

MIT — Copyright Luis Recalde 2026. Ver [LICENSE](LICENSE).
