# Penetration test report: DVWA (low security)

**Tester:** Farhan Hussain
**Target:** Damn Vulnerable Web Application, running locally at `http://192.168.122.1:4280`
**Security level:** Low (set deliberately, to establish a baseline)
**Date:** 2026-11-03

## 1. Scope and rules of engagement

- **In scope:** the DVWA instance above, running in a Docker container on a lab host I own. The app is designed to be attacked and has no real users or data.
- **Out of scope:** everything else on the lab network.
- **Method:** black-box at first (unauthenticated recon), then authenticated testing with the app's default low-privilege demo account. No automated exploitation tools were used against the live app beyond the two tools in this portfolio ([reconkit](../../reconkit), [apicheck](../../apicheck)); the vulnerability findings below were confirmed by hand with minimal, safe proof-of-concept requests.
- **Goal:** identify the vulnerabilities DVWA is built to demonstrate, show real evidence for each, and give the fix — the same shape as a client report.

## 2. Summary

| # | Finding | Severity |
|---|---|---|
| 1 | SQL injection in the "User ID" search (`/vulnerabilities/sqli/`) | High |
| 2 | OS command injection in the ping tool (`/vulnerabilities/exec/`) | High |
| 3 | Reflected cross-site scripting in the XSS demo (`/vulnerabilities/xss_r/`) | Medium |
| 4 | No security headers (CSP, X-Frame-Options, HSTS, etc.) | Medium |
| 5 | Session cookie missing HttpOnly and SameSite | Medium |
| 6 | Server and framework versions disclosed (Apache 2.4.69, PHP 8.5.12) | Low |

Findings 4–6 were produced automatically by [apicheck](../../apicheck) (`apicheck_dvwa.md`, included in this folder); 1–3 were confirmed by hand because they need an authenticated session and app-specific payloads, which a generic checker shouldn't guess at.

## 3. Findings

### 3.1 SQL injection — High

**Where:** `GET /vulnerabilities/sqli/?id=<input>&Submit=Submit`

The page builds a SQL query directly from the `id` parameter with no sanitisation. Submitting a numeric ID returns one row; submitting a classic boolean payload returns every row in the table.

**Proof of concept:**
```
GET /vulnerabilities/sqli/?id=1' OR '1'='1&Submit=Submit
```
Result: the page returned all 5 user records instead of 1, confirming the query is not parameterised.

**Impact:** an attacker can read, and on a writable account modify, any data the database user can reach — here, every user's first and last name, and in a real app potentially passwords, emails or payment data.

**Fix:** use parameterised queries (prepared statements) everywhere user input reaches SQL. Never build queries with string concatenation. Apply least privilege to the database account the app uses.

### 3.2 OS command injection — High

**Where:** `POST /vulnerabilities/exec/` (`ip` parameter)

The page is meant to ping an IP address, but passes the input to a shell command without filtering shell metacharacters.

**Proof of concept:**
```
ip=127.0.0.1 && id
```
Result: the response included `uid=33(www-data) gid=33(www-data) groups=33(www-data)`, i.e. the output of an injected `id` command, not just the ping result.

**Impact:** full command execution as the web server user. From here an attacker can read application source and config (database credentials, secrets), pivot to other hosts, or attempt privilege escalation.

**Fix:** never pass user input to a shell. If a system command is unavoidable, use an API that takes arguments as a list (no shell interpretation) and validate the input against a strict allow-list (e.g. a valid IPv4/IPv6 pattern).

### 3.3 Reflected cross-site scripting — Medium

**Where:** `GET /vulnerabilities/xss_r/?name=<input>`

The `name` parameter is echoed into the page without HTML-encoding.

**Proof of concept:**
```
GET /vulnerabilities/xss_r/?name=<script>alert(1)</script>
```
Result: the raw `<script>` tag appeared unescaped in the response body.

**Impact:** an attacker who can get a victim to click a crafted link can run arbitrary JavaScript in the victim's session — stealing cookies, performing actions as the victim, or redirecting them.

**Fix:** HTML-encode all user input on output (DVWA's "medium"/"high" levels show the fix), and add a Content-Security-Policy as defence in depth.

### 3.4 Missing security headers — Medium

Confirmed automatically with apicheck; see `apicheck_dvwa.md`. The app sends no `Content-Security-Policy`, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy` or (on a TLS deployment) `Strict-Transport-Security`.

**Impact:** these headers are defence in depth against XSS, clickjacking and MIME-sniffing attacks. Their absence doesn't cause a breach by itself, but removes a layer that would have reduced the impact of 3.3.

**Fix:** add the headers above at the web server or framework level.

### 3.5 Session cookie missing HttpOnly / SameSite — Medium

`PHPSESSID` is set without `HttpOnly` or `SameSite`, and the app's own `security` cookie is missing the same flags.

**Impact:** without `HttpOnly`, a successful XSS (see 3.3) can read the session cookie directly, turning a script-injection bug into full account takeover. Without `SameSite`, the cookie is sent on cross-site requests, weakening CSRF defences.

**Fix:** set `HttpOnly`, `Secure` (once served over HTTPS) and `SameSite=Lax` or `Strict` on session cookies.

### 3.6 Version disclosure — Low

The `Server` and `X-Powered-By` headers reveal `Apache/2.4.69 (Debian)` and `PHP/8.5.12`.

**Impact:** low on its own, but it tells an attacker exactly which known CVEs to try first.

**Fix:** turn off `ServerTokens`/`expose_php`, or set generic values.

## 4. Overall risk

The combination of 3.1–3.3 is the real risk: unauthenticated or low-privilege input reaches a database query, a shell, and an HTML page without being checked anywhere. On a real application this chain would be rated **Critical**, since 3.2 alone gives code execution. 3.4–3.6 are not exploitable by themselves but make 3.1–3.3 worse.

## 5. Recommendations, in priority order

1. Fix the command injection first (3.2) — it is the most severe and the simplest to fix (stop calling a shell).
2. Parameterise all SQL (3.1).
3. Encode all output (3.3), then add CSP as a backstop.
4. Add the missing headers and cookie flags (3.4, 3.5).
5. Hide version strings (3.6).

## Appendix: raw apicheck output

See [`apicheck_dvwa.md`](apicheck_dvwa.md) for the full automated scan this report draws findings 4–6 from, and [`recon_dvwa.md`](recon_dvwa.md) for the port and service scan.
