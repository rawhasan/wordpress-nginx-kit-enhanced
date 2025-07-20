# WordPress Nginx Kit – Purpose & Overview

The **WordPress Nginx Kit** is a modular, production-ready configuration toolkit built to host **single-site or multisite WordPress** installations on an **Nginx web server** with security, performance, and maintainability as top priorities.

---

## 🧩 Structure and Purpose

This kit is structured around clean separation of configuration concerns using the `include` directive. It consists of:

### `global/server/`
Modular config files that can be reused across multiple sites:

- **`defaults.conf`**  
  Bundles the three core includes: `exclusions.conf`, `security.conf`, and `static-files.conf` for convenience.

- **`exclusions.conf`**  
  Blocks access to hidden files, logs, configs, and disallows PHP in `/uploads`.

- **`security.conf`**  
  Adds security headers like:
  - `Content-Security-Policy`
  - `Referrer-Policy`
  - `Permissions-Policy`
  - `X-Frame-Options`, `X-XSS-Protection`, `X-Content-Type-Options`

- **`ssl.conf`**  
  Enforces modern SSL/TLS best practices:
  - TLSv1.2+ only
  - Secure ciphers
  - HSTS with optional subdomain coverage
  - `ssl_dhparam` support for forward secrecy

- **`gzip.conf`**  
  Enables efficient compression for most MIME types using gzip with sensible buffer and compression settings.

- **`static-files.conf`**  
  Caches static content (images, fonts, JS, CSS) for up to 1 year. Prevents logging for static hits and adds CORS for fonts.

- **`limits.conf`**  
  Sets timeout values and `client_max_body_size` for performance and upload control.

- **`mime-types.conf`**  
  Maps file extensions to correct MIME types to ensure proper `Content-Type` headers.

- **`fastcgi-cache.conf`**  
  Implements FastCGI full-page caching for WordPress with:
  - skip rules for POST, query strings, admin URLs
  - exclusions for logged-in users and carts
  - fallback behavior using `fastcgi_cache_use_stale`
  - debug header `Fastcgi-Cache`

- **`php-pool.conf`**  
  Defines upstream PHP socket (e.g. `php8.3`) and allows for future switching by overriding the `map`.

---

### `sites-available/`
Server blocks that can be symlinked from `sites-enabled/`, supporting:

- **Single-site WordPress**
- **Multisite (subdomain or subdirectory)**
- **No SSL, SSL, or SSL + caching**
- Upstream selector integration

---

## ✅ Benefits

- Modular and reusable per-site includes
- Security headers and file-level hardening baked in
- Production caching and compression strategies
- Clear default behaviors with override flexibility
- Logging minimized for safe/static requests
- TLS and HSTS implemented properly
- Designed specifically for WordPress nuances

---

## 🛠️ Use Case

This kit is ideal for:

- Developers hosting multiple WordPress sites
- System admins setting up new servers
- Users migrating from Apache to Nginx
- Anyone needing pre-audited Nginx configs with modern standards

---
