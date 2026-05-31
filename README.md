# WebGuard

Revisá si tu sitio web está seguro — sin necesitar conocimientos técnicos.

WebGuard revisa tu sitio antes de publicarlo y te avisa si hay algo que puede poner en riesgo a tus visitantes, tus datos o tu negocio. No hace falta saber de programación ni de ciberseguridad: WebGuard detecta los problemas y te dice exactamente qué corregir, con palabras claras.

Es un skill para Claude Code: se instala una vez y lo usás cuando querés.

---

## Instalación

```bash
git clone https://github.com/luis-recalde/webguard ~/.claude/skills/webguard
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

Para cualquier persona que tiene un sitio web y quiere publicar tranquila, sabiendo que su sitio no tiene problemas de seguridad. No necesitás saber qué es un header HTTP ni qué significa OWASP.

Instalás WebGuard, lo ejecutás, y te dice si hay algo que arreglar — en español, con instrucciones concretas. Sin tecnicismos innecesarios.

---

## Licencia

MIT — Copyright Luis Recalde 2026. Ver [LICENSE](LICENSE).

**Autor:** info@luisrecalde.com
