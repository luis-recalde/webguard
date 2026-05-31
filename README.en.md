# WebGuard

Automatic security audit for Next.js websites, integrated directly into Claude Code.

WebGuard analyzes your site before deployment and detects real vulnerabilities: secrets exposed in code, misconfigured HTTP headers, dependencies with known flaws, insecure forms, and OWASP Top 10 attack vectors. When done, it delivers a clear report with the specific fixes needed.

---

## Installation

```bash
git clone https://github.com/luis-recalde/webguard ~/.claude/skills/webguard
```

One command. No additional setup.

---

## How to use it

Once installed, you have two ways to trigger it:

**With the direct command:**
```
/webguard
```

**With natural language** — just tell Claude:
- *"Audit the security of my site"*
- *"Is the site ready to publish?"*
- *"Review security before deploying"*
- *"Is this safe?"*

WebGuard also activates automatically when you add new integrations (payments, forms, third-party services) or before any production deployment.

---

## What it analyzes

### HTTP security headers
Verifies that `next.config.js` includes the headers modern browsers require. If any are missing, WebGuard adds them with the correct configuration: `X-Frame-Options`, `Content-Security-Policy`, `X-Content-Type-Options`, `Referrer-Policy`, and `Permissions-Policy`.

### Hardcoded secrets
Scans the project with over 60 patterns to detect API keys, access tokens, passwords, private keys, and database credentials exposed in source code. If anything is found, the deployment is blocked.

### Vulnerable dependencies
Runs `npm audit` and interprets results with judgment: it distinguishes between vulnerabilities that pose real production risk and those that only affect the development environment.

### `.gitignore` configuration
Verifies that environment variable files (`.env`, `.env.local`, `*.pem`) are properly excluded from the repository. If entries are missing, it adds them.

### External links
Detects links without HTTPS and missing security attributes (`rel="noopener noreferrer"`) that can expose visitors to redirect attacks.

### Sensitive data in the client bundle
Checks that no secret API keys or private tokens are exposed in the JavaScript code that visitors' browsers download.

### Forms and submissions
Verifies that form endpoints use HTTPS and that contact data travels securely.

### OWASP Top 10
Covers the attack vectors relevant to Next.js sites: Broken Access Control, Cryptographic Failures, Injection, Security Misconfiguration, Vulnerable Components, Software Integrity Failures, and SSRF.

---

## Who it's for

WebGuard is designed for website owners and developers who want to publish with the confidence that their site is secure, without needing to be cybersecurity experts.

The analysis is automatic. The report is direct. The fixes are concrete.

If something is wrong, WebGuard says so clearly and corrects it before the site goes live.

---

## License

MIT — Copyright Luis Recalde 2026. See [LICENSE](LICENSE).
