# Enhancements implemented on the original kit

### After apply:

```
sudo nginx -t 
sudo systemctl reload nginx
```
```
sudo systemctl status nginx
```

### Scan sites after applying:
- https://securityheaders.com/
- https://developer.mozilla.org/en-US/observatory
- https://www.ssllabs.com/

## 🛡️ Security Headers Configuration for Nginx

**File edited:**
`global/server/security.conf`

These headers improve browser security and help mitigate attacks like XSS, data injection, and feature abuse.


### 1. Content-Security-Policy (CSP)

`add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' https://www.googletagmanager.com https://www.google-analytics.com https://cdnjs.cloudflare.com https://nijhoom.b-cdn.net; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdnjs.cloudflare.com https://nijhoom.b-cdn.net; font-src 'self' https://fonts.gstatic.com https://nijhoom.b-cdn.net; img-src 'self' data: https://nijhoom.b-cdn.net https://www.google-analytics.com; connect-src 'self' https://nijhoom.com https://nijhoom.b-cdn.net https://www.google-analytics.com; frame-ancestors 'self'; form-action 'self';" always;`

Purpose: Prevents loading of external/untrusted scripts, styles, fonts, and connections unless explicitly allowed.


### 2. Referrer-Policy

`add_header Referrer-Policy "strict-origin-when-cross-origin" always;`

Purpose: Prevents leaking full URL referrer information to third-party domains.


### 3. Permissions-Policy

`add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;`

Purpose: Disables browser access to geolocation, mic, and camera unless explicitly allowed.






## 1. File: `nginx.conf`

**Path:** `wordpress-nginx-kit/nginx.conf`

- **Added `server_tokens off;`**  
  Enhances security by hiding the Nginx version number in HTTP response headers.

These changes were applied inside the `http {}` block for global effectiveness across all hosted sites.


## 2. File: `global/gzip.conf`

**Path:** `wordpress-nginx-kit/global/gzip.conf`

- ✅ Removed outdated or duplicate settings:
  - `gzip_comp_level`
  - `gzip_buffers`
  - simplified `gzip_types` line
  - associated inline comments

- ✅ Added updated settings with fresh comments:
  - `gzip_comp_level 6;` — balanced for performance and CPU usage
  - `gzip_buffers 16 8k;` — defines buffer size for compressed data

This ensures consistent and optimized gzip compression with no redundant declarations.
