# Security requirements

Numeric values (durations, limits) are in limits.md and referenced here as [L: ...].

## 1. Passwords
- Stored only as Argon2id hashes (OWASP recommendation). Plain passwords are never stored or logged.
- Requirements: 8 to 128 characters [L: password]; the password must not match the email or nickname. No mandatory "uppercase + digit + special character" rules: length matters more than complexity (NIST recommendation).
- A strength indicator is shown while typing.

## 2. Sessions
- Server-side sessions stored in PostgreSQL in our own table (see database.md). The cookie holds a random 256-bit session token; the database stores only its SHA-256 hash.
- Cookie: `HttpOnly`, `Secure`, `SameSite=Lax`.
- A new session is created on every sign-in (session fixation protection).
- Lifetime: extended on activity, but not beyond an absolute limit [L: sessions].
- The user sees the list of active sessions (device, IP address, last activity) and can end any of them.
- Sessions are terminated on: sign-out, ending a session from the device list, "Sign out on all devices", password change or recovery (except the current one on change), block, account deletion.
- CSRF protection: Spring Security CSRF token (stored in a cookie) for all state-changing requests.

## 3. Email tokens
Email confirmation, password recovery, email change, password change, account deletion.
- Random, 256 bits, from a cryptographically secure generator.
- Only the token hash (SHA-256) is stored in the database.
- Single-use: deleted after use.
- Limited lifetime [L: tokens].
- A new token of the same type invalidates the previous one.
- Passed after `#` in the URL and sent to the backend in a POST request body (see ui.md).

## 4. Brute force and abuse protection
- Rate limits per IP and per account: sign-in, registration, password recovery, email resend, sharing by email, public link creation [L: rate limits].
- After a series of failed sign-ins to an account — an increasing delay before the next attempt, without locking the account completely (otherwise an attacker could lock anyone out).

## 5. Hiding account existence
- Sign-in: one error "Invalid login or password" for all cases.
- Password recovery and sharing by email: the same response whether or not the email exists.
- Registration reports that the email or nickname is taken. This is a deliberate usability trade-off, mitigated by rate limits.

## 6. Authorization
- Every request to a file, folder, link or access entry checks the user's right to that object: owner, granted access or public link.
- A missing object and someone else's object return the same `404`, so the existence of other users' objects cannot be detected.
- Object IDs are UUIDs.
- `/api/v1/admin/**` endpoints are available only with the `ADMIN` role, checked on the backend.
- Unconfirmed users, blocked users and users over quota are denied forbidden actions on the backend, not only by hidden buttons.
- Every endpoint has tests for access to other users' objects.

## 7. Files
- The bucket is private; there is no direct public access to objects.
- The object key in storage is a UUID; the file name is stored only in the database.
- Upload and download via presigned URLs with a short lifetime [L: presigned URLs].
- An upload URL is issued for a specific key and declared size. After upload, the backend checks the actual object size; if it does not match or exceeds the quota, the object is deleted.
- Incomplete uploads are cleaned up automatically [L: incomplete upload retention].
- Downloads are served with the `Content-Disposition: attachment` header.
- In-browser preview only for safe types (see functional.md, 2.5). SVG and HTML are never rendered in the browser: they can contain executable code.
- Text files are shown as text, not as HTML.
- Files are served from the storage domain, not the site domain: even a malicious file cannot access the user's session.

## 8. Encryption
- All traffic over HTTPS only (TLS 1.2+), HTTP redirects to HTTPS, HSTS enabled.
- The bucket is encrypted with a Yandex KMS key.
- Database backups are stored in an encrypted bucket.

## 9. Web security
- Frontend and API are served from the same domain (Nginx proxies `/api`), so CORS is not needed and is disabled.
- Headers: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `frame-ancestors 'none'` (the site cannot be embedded in other pages).
- All input is validated on the backend (Bean Validation). Database queries are parameterized only (JPA).
- Errors do not expose internals: no stack traces or SQL in responses.

## 10. Secrets
- Only in environment variables on the server or in Yandex Lockbox. The repository contains only `.env.example`.
- CI checks for accidentally committed secrets (gitleaks).
- The application's service account in Yandex Cloud has minimal required permissions.

## 11. Infrastructure
- SSH by key only; root login disabled.
- Only ports 22, 80, 443 are open to the internet (Yandex Cloud security groups).
- PostgreSQL, Prometheus and Grafana are not reachable from the internet. Grafana via an SSH tunnel or behind authentication.
- Containers run as non-root on minimal base images.
- OS security updates are installed automatically (unattended-upgrades).
- Application Docker images are rebuilt in CI on a schedule to pick up base image fixes. PostgreSQL, Nginx and other service version upgrades are manual, via pull request.

## 12. Dependencies
- Dependabot for dependency updates.
- Docker image vulnerability scanning (Trivy) in CI.

## 13. Email
- SPF, DKIM and DMARC are configured for the domain so emails do not land in spam and cannot be spoofed on behalf of the service.

## 14. Payments
- Payment notifications are verified: the payment status is re-checked with a request to the payment provider's API.
- Notification handling is idempotent: a repeated notification does not add space a second time.

## 15. Activity log and logs
- The user and admin activity log (see functional.md, 7) is stored in the database [L: log retention].
- Passwords, tokens, cookies and presigned URLs never appear in logs.

## 16. Personal data
- Only necessary data is collected: email, nickname, IP addresses (activity log, sessions), browser user agent (sessions).
- Data is stored in Russia (Yandex Cloud).
- The site has a privacy policy and terms of service.
- Requirements of Federal Law 152-FZ for a non-commercial pet project — to be clarified separately.

## Later (not MVP)
- Two-factor authentication for admins, then for users.
