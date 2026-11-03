# apicheck report: http://192.168.122.1:4280

Requests sent: 28. Findings: 9.

Findings are indicators to verify by hand, not proof of exploitability.

## [MEDIUM] Content-Security-Policy missing

- Check: headers
- Detail: Response to GET / has no content-security-policy header.
- Fix: Add a CSP that limits script and frame sources.

## [MEDIUM] Cookie 'PHPSESSID' lacks HttpOnly, SameSite

- Check: cookies
- Detail: PHPSESSID=9149861202287fa7ffbd6898a43717ff; expires=Wed, 04 Nov 2026 10:46:37 GMT; Max-Age=86400; path=/; domain=192.168
- Fix: Set HttpOnly, Secure (on HTTPS) and SameSite on session cookies.

## [LOW] X-Content-Type-Options missing

- Check: headers
- Detail: Response to GET / has no x-content-type-options header.
- Fix: Send X-Content-Type-Options: nosniff.

## [LOW] Clickjacking protection missing

- Check: headers
- Detail: Response to GET / has no x-frame-options header.
- Fix: Send X-Frame-Options: DENY or a CSP frame-ancestors rule.

## [LOW] Referrer-Policy missing

- Check: headers
- Detail: Response to GET / has no referrer-policy header.
- Fix: Send Referrer-Policy: strict-origin-when-cross-origin.

## [LOW] Cookie 'security' lacks HttpOnly, SameSite

- Check: cookies
- Detail: security=medium; path=/
- Fix: Set HttpOnly, Secure (on HTTPS) and SameSite on session cookies.

## [INFO] Version disclosed in header

- Check: headers
- Detail: server: Apache/2.4.69 (Debian)
- Fix: Hide or genericise version strings.

## [INFO] Version disclosed in header

- Check: headers
- Detail: x-powered-by: PHP/8.5.12
- Fix: Hide or genericise version strings.

## [INFO] robots.txt lists paths worth a look

- Check: exposure
- Detail: GET /robots.txt returned 200 (25 bytes).
- Fix: Remove the file from the web root or block access to it.
