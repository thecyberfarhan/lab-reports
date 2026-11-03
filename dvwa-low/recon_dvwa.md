# Recon report: 192.168.122.1

Resolved address: `192.168.122.1`

## Open ports

| Port | Service | Banner |
|---|---|---|
| 4280 | unknown | - |
| 8080 | http-alt | - |

## Web services

- http://192.168.122.1:4280: HTTP 200, title "Login :: Damn Vulnerable Web Application (DVWA)", server `Apache/2.4.69 (Debian)`, missing headers: Strict-Transport-Security, Content-Security-Policy, X-Content-Type-Options, X-Frame-Options, Referrer-Policy, Permissions-Policy
- http://192.168.122.1:8080: HTTP 200, title "SearXNG", server `granian`, missing headers: Strict-Transport-Security, Content-Security-Policy, X-Frame-Options, Permissions-Policy
