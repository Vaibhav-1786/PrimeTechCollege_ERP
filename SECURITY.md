# Security Policy

PrimeTechCollege_ERP handles student and faculty records, fee payments, results and private messages. Security reports are taken seriously.

## Supported versions

Only the latest commit on the `main` branch receives security fixes.

## Reporting a vulnerability

**Please do not open a public issue for security problems.**

1. Use GitHub's private reporting: **Security → Report a vulnerability** on this repository (preferred), or
2. Email the maintainer, Vaibhav Chauhan, at "vaibhavchauhan1786@gmail.com".

Please include a description, impact, steps to reproduce (endpoint, role, request/response) and a suggested fix if you have one. You can expect an acknowledgement within **7 days** and a status update within **14 days**. Please allow reasonable time for a fix before public disclosure.

### In scope
Role bypass between admin / faculty / student, access to another student's results, fees or messages, authentication or session-token flaws, SQL injection, Telegram webhook spoofing, payment/receipt tampering, secrets exposure.

### Out of scope
Issues that exist only because development defaults were left in production (see checklist), denial of service by volume, social engineering, and flaws in third-party services (Razorpay, OpenRouter, Telegram, hosting).

## Security measures in the project

- Role-based access (admin / faculty / student), checked in the frontend (`rbac.js`) and in the PHP endpoints.
- Admin sessions are server-signed using `APP_TOKEN_SECRET`.
- The OpenRouter API key stays server-side; the AI proxy limits conversation length and message/payload size.
- Telegram webhook requests are validated against `TELEGRAM_WEBHOOK_SECRET`.
- PDO is used for database access in the PHP API.
- `backend/.env` and `frontend/dist/` are gitignored.

## Production deployment checklist

The repository ships with **development defaults**. Before going live:

- [ ] **Change the static admin credentials.** `STATIC_ADMIN_EMAIL` / `STATIC_ADMIN_PASSWORD` are constants in `backend/config/helpers.php`; move them into `.env` and use a strong password.
- [ ] Generate a long random `APP_TOKEN_SECRET` (`php -r "echo bin2hex(random_bytes(32));"`).
- [ ] **Enforce student and faculty authorisation in PHP.** Their sessions are managed client-side, so every endpoint must verify identity and ownership on the server — never rely on `AuthContext` or `rbac.js` alone.
- [ ] Set `TELEGRAM_WEBHOOK_SECRET` and keep `TELEGRAM_BOT_TOKEN` private.
- [ ] Set `CLIENT_URL` / `EXTRA_CLIENT_URLS` to your real origins only.
- [ ] Switch `frontend/src/utils/api.js` base URLs and Razorpay to the correct environment; use Live Razorpay button IDs only after testing, and confirm payments server-side.
- [ ] Use a dedicated MySQL user with least privilege, not `root`.
- [ ] Serve everything over HTTPS; run PHP through PHP-FPM and `backend/server.js` under PM2/systemd.
- [ ] Do not use `php -S` in production.
- [ ] Keep `backend/.env` out of git; rotate any secret that was ever committed.
- [ ] Protect student data according to applicable law (e.g. India's DPDP Act 2023).

## Known limitations

- Admin sign-in uses static credentials rather than a database user.
- Student/faculty session state lives in the browser (`localStorage`), so it is readable by any script on the page; a strict Content-Security-Policy is recommended.
- No automated test suite is documented yet.
- The AI assistant sends student prompts (with course and semester context) to a third-party provider (OpenRouter).
