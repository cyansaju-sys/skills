---
name: security-audit
description: >
  Audits code, configuration, and dependencies for security vulnerabilities, prioritizes
  findings by severity, and applies fixes (OWASP, CWE, OWASP MASVS for mobile). Use it when
  the user asks to review security, find vulnerabilities, harden code, check for leaked
  secrets, or audit dependencies (Spanish: "ciberseguridad", "seguridad", "vulnerabilidades",
  "auditoría de seguridad", "revisa la seguridad", "hardening", "secretos expuestos",
  "es seguro este código"). Covers web backends, frontends, APIs, and Android apps.
  Defensive use only.
---

# Skill: Security Audit

## Purpose

Find, prioritize, and fix security vulnerabilities in the user's code, explaining the real
risk of each finding and delivering a concrete fix.

Write all explanations in the user's language.

## Scope and Ethics

- Only audit code and systems the user owns or is authorized to test.
- Defensive focus: explain the risk and fix it. Do not build weaponized exploits against
  third-party systems. A minimal malicious input inside a regression test is fine.

---

## Process

### 1. Define the Scope

- **What:** a file, module, diff, or the whole repository
- **Stack:** language, framework, runtime, deployment type (web, API, mobile, CLI)
- **Attack surface:** entry points (HTTP routes, forms, file uploads, deep links, exported
  Android components, queues), trust boundaries, and sensitive data handled
  (credentials, PII, payments)

For large codebases, start with entry points, authentication, and authorization.

### 2. Automated Checks (if available)

Prefer tools already configured in the project. **Ask for confirmation before installing
any tool.** Treat tool output as leads: verify each finding manually and discard false positives.

| Check | Tools |
|---|---|
| Leaked secrets | gitleaks, trufflehog |
| Vulnerable dependencies | `npm audit`, `pip-audit`, osv-scanner, OWASP Dependency-Check |
| Static analysis (SAST) | semgrep, bandit (Python), eslint-plugin-security, `./gradlew lint`, MobSF (Android) |

### 3. Manual Review Checklist

| Category (CWE) | Look for | Fix |
|---|---|---|
| SQL/NoSQL injection (CWE-89) | Queries built by string concatenation/interpolation | Parameterized queries, ORM bindings |
| Command injection (CWE-78) | `exec`, `eval`, `Runtime.exec`, `shell=True` with user input | Avoid the shell, argument arrays, allowlists |
| XSS (CWE-79) | `innerHTML`, `dangerouslySetInnerHTML`, `v-html`, unescaped templates | Contextual output encoding, DOMPurify, CSP |
| Missing authorization / IDOR (CWE-862, CWE-639) | Resources fetched by ID without an ownership check; unprotected routes | Server-side authorization check on every request |
| Broken authentication (CWE-287) | No rate limiting, unverified JWTs or `alg: none`, session fixation | Verified signatures, rate limiting, session rotation on login |
| Weak password storage (CWE-916) | MD5/SHA-1/unsalted hashes | Argon2id or bcrypt |
| CSRF (CWE-352) | State-changing GET requests, cookies without `SameSite`, no CSRF token | CSRF tokens, `SameSite`, non-GET for mutations |
| SSRF (CWE-918) | Server fetches user-supplied URLs | Host allowlist, block internal/metadata IP ranges |
| Hardcoded secrets (CWE-798) | API keys, passwords, tokens in code or config | Environment variables / secret manager |
| Weak cryptography (CWE-327, CWE-338) | ECB mode, static IVs, `Math.random()` for tokens | AES-GCM, CSPRNG (`crypto.randomUUID`, `secrets`) |
| Insecure deserialization (CWE-502) | `pickle`, `ObjectInputStream`, `yaml.load` on untrusted data | Safe formats (JSON), `yaml.safe_load` |
| Path traversal / uploads (CWE-22, CWE-434) | User input in file paths; no type/size checks | Canonicalize and verify base dir; validate type and size |
| Misconfiguration | Debug mode in production, `CORS *` with credentials, missing security headers, stack traces in responses | Environment-specific config, strict CORS, security headers |
| Sensitive data in logs (CWE-532) | Tokens, passwords, PII written to logs | Redact or omit |

### 4. Android-Specific Checks (OWASP MASVS)

- `android:exported="true"` components without permissions; intent redirection
- `android:debuggable`, `android:allowBackup="true"`, `usesCleartextTraffic` in release builds
- Network Security Config: no user-installed CAs in release; consider certificate pinning for high-risk apps
- Secrets in `BuildConfig`, `strings.xml`, or code — they are extractable from the APK; move them to a backend
- Sensitive data in plaintext `SharedPreferences` or files → encrypt with keys held in the Android Keystore
- WebView: `setJavaScriptEnabled(true)` + `addJavascriptInterface` with untrusted content; `setAllowFileAccess(true)`
- `PendingIntent` with implicit intents and `FLAG_MUTABLE` → prefer `FLAG_IMMUTABLE`
- Deep links: validate every parameter; verify App Links
- `Log.*` with sensitive data in release; R8/minification disabled

### 5. Report Findings

Order findings by severity. Use this format for each one:

```
### [SEVERITY] Short title
- Location: `path/file.ext:line`
- Category: CWE-XXX
- Risk: concrete attack scenario and impact
- Fix: code showing only the changed part
```

| Severity | Criteria |
|---|---|
| Critical | Remotely exploitable without authentication: RCE, full data leak, auth bypass |
| High | Significant data exposure or privilege escalation with some preconditions |
| Medium | Requires user interaction, specific conditions, or has limited impact |
| Low | Defense-in-depth or hardening improvement |

End with a summary table (count per severity) and state explicitly what was **not** reviewed.

### 6. Apply Fixes

- Fix Critical and High first; ask before large refactors.
- Add a regression test with the malicious input when feasible.
- Never "fix" by disabling validation or suppressing tool warnings without justification.
- **Leaked secrets:** removing them from code is not enough — they remain in git history.
  Tell the user to revoke and rotate them. Rewriting git history is destructive: only with explicit confirmation.

---

## Integration with Other Skills

- **clean-code:** new code must also pass the checklist in section 3.
- **migrations:** after a migration, verify that security controls (authentication, validation,
  CSRF, headers, CORS) were carried over — frameworks have different secure defaults.
- **browser-testing:** use it to confirm XSS/CSRF/access-control fixes in a real browser.
