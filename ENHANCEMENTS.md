# Enhancements implemented on the original kit

### After apply:

```
sudo nginx -t 
sudo systemctl reload nginx
```
```
sudo systemctl status nginx
```

## Security Headers Configuration for Nginx

**File edited:**
`global/server/security.conf`

These headers improve browser security and help mitigate attacks like XSS, data injection, and feature abuse.


### 1. Content-Security-Policy (CSP)

`add_header Content-Security-Policy "default-src 'self'; script-src 'self' https://www.google-analytics.com https://www.googletagmanager.com https://cdnjs.cloudflare.com https://nijhoom.b-cdn.net; style-src 'self' 'unsafe-inline' https://cdnjs.cloudflare.com https://fonts.googleapis.com https://nijhoom.b-cdn.net; font-src 'self' https://fonts.gstatic.com https://nijhoom.b-cdn.net; img-src 'self' data: https://www.google-analytics.com https://nijhoom.b-cdn.net; connect-src 'self' https://www.google-analytics.com https://nijhoom.com https://nijhoom.b-cdn.net; frame-ancestors 'self'; form-action 'self';" always;`

Purpose: Prevents loading of external/untrusted scripts, styles, fonts, and connections unless explicitly allowed.


### 2. Referrer-Policy

`add_header Referrer-Policy "strict-origin-when-cross-origin" always;`

Purpose: Prevents leaking full URL referrer information to third-party domains.


### 3. Permissions-Policy

`add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;`

Purpose: Disables browser access to geolocation, mic, and camera unless explicitly allowed.
