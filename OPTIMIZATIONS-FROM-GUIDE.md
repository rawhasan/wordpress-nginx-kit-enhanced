# Nginx Optimizations from the Guide

## Basic Optimizations

Open the Nginx Configuration file:
```
sudo nano /etc/nginx/nginx.conf
```



### User & Worker Process

Get the number of CPU cores your server has available:

```
grep processor /proc/cpuinfo | wc -l
```

Get your server’s open file limit:

```
ulimit -n
```

The `worker_processes` directive determines how many workers to spawn per server. The general rule of thumb is to set this to the number of CPU cores your server has available. In my case, this is `1`.

### Events Block

- `worker_connections` should be set to your server’s open file limit. This tells Nginx how many simultaneous connections can be opened by each worker_process. Therefore, if you have two CPU cores and an open file limit of 1024, your server can handle 2048 connections per second. (🔲 Apply per site basis)

- The `multi_accept` directive should be uncommented and set to `on`. This informs each `worker_process` to accept all new connections at a time, opposed to accepting one new connection at a time. (🔲 Apply per site basis)



### Http Block

- In `http` block. The first directive to add is `keepalive_timeout`. The `keepalive_timeout` determines how many seconds a connection to the client should be kept open before it’s closed by Nginx. This directive should be lowered, as you don’t want idle connections sitting there for up to 75 seconds if they can be utilized by new clients. I have set mine to `15`. You can add this directive just above the `sendfile on;` directive. (✔️ Already available in `global/limits.conf`)

- For security reasons, you should uncomment the `server_tokens` directive and ensure it is set to `off`. This will disable emitting the Nginx version number in error messages and response headers. (✔️ Added in `global/http.conf`)



### Gzip Compression (Made additional improvements by ChatGPT)

- Uncomment the `gzip_proxied` directive and set it to `any`, which will ensure all proxied request responses are gzipped. (✔️ Already available in `global/gzip.conf`)

- Uncomment the `gzip_comp_level` and set it to a value of `5` (ChatGPT Suggests `6` as Standard). This controls the compression level of a response and can have a value in the range of 1 – 9. Be careful not to set this value too high, as it can have a negative impact on CPU usage. (✔️ Already available in `global/gzip.conf`. Value updated to `6`)

- Uncomment the `gzip_types` directive, leaving the default values in place. This will ensure that JavaScript, CSS, and other file types are gzipped in addition to the HTML file type which is always compressed by the gzip module. (✔️ Already available in `global/gzip.conf`)



**Restart Nginx**
```
sudo nginx -t
sudo service nginx restart
```
```






```
## WordPress maximum upload size

You should adjust your `php.ini` file to increase the WordPress maximum upload size. Both this and the `client_max_body_size directive` within Nginx must be changed for the new maximum upload limit to take effect.

I chose a value of `64m` but you can increase it if you run into issues uploading large files.

**open the Nginx configuration file:**
```
sudo nano /etc/nginx/nginx.conf
```

Underneath `server_tokens` add the following line to set the maximum upload size you require in the WordPress Media Library:

```client_max_body_size 64m;``` (✔️ Already available in `global/limits.conf`)

**Open your php.ini file:** 
```
sudo nano /etc/php/8.3/fpm/php.ini
```

Change the following lines to match the value you assigned to the client_max_body_size directive when configuring Nginx: (🔲 Apply after installing PHP)
```
upload_max_filesize = 64M
```
```
post_max_size = 64M
```
```






```
## OPcache
Enable the OPcache file override setting. When this setting is enabled, OPcache will serve the cached version of PHP files without checking if the file has been modified on the file system, resulting in improved PHP performance.

Open your php.ini file:
```
sudo nano /etc/php/8.3/fpm/php.ini
```

Hit `CTRL + W` and type `file_override` to locate the line we need to update. Now uncomment it (remove the semicolon) and change the value from zero to one: (🔲 Apply after installing PHP)
```
opcache.enable_file_override = 1
```
```






```
## Run WordPress as the Server User
Changing the user causes trouble. DO NOT CHANGE. Run WordPress as `www-data` and add server user to the group (See Below)

[Steps details and script to change user and revert.](https://github.com/rawhasan/wordpress-lemp-server/tree/main/wordpress-user-change)
```






```
## ✅ Add the Server User to the `www-data` Group

This is the safest and most common way to give your server user (e.g., for SSH or deployments) access to the WordPress directory owned by `www-data`.

### Steps:

---

### 1. Check the Ownership of the WordPress Directory

```bash
ls -ld /var/www/html
```

**Expected output:**

```text
drwxr-xr-x  7 www-data www-data 4096 Jul 21 12:34 /var/www/html
```

---

### 2. Add Your User to the `www-data` Group

Replace `youruser` with your actual username:

```bash
sudo usermod -aG www-data youruser
```

> ⚠️ **Note:** You must log out and log back in (or reboot) for the group change to take effect.

---

### 3. Set Proper Permissions (Optional but Recommended)

Make directories and files group-writable:

```bash
sudo find /var/www/html -type d -exec chmod 775 {} \;
sudo find /var/www/html -type f -exec chmod 664 {} \;
```

---

### 4. Set the `setgid` Bit (So New Files Inherit the Group)

```bash
sudo chmod g+s /var/www/html
```
```






```
## SSL Hardening
### Strict Transport Security
Although your site is configured to only handle HTTPS traffic via your SSL certificate from Let’s Encrypt, it still allows the client to attempt further HTTP connections. Adding the `Strict-Transport-Security` header to the server response will ensure all future connections enforce HTTPS. (✔️ Uncommented the subdomain option in `global/server/ssl.conf`).

Now let’s update our SSL configuration as per the recommendations of Mozilla’s SSL Configuration Generator on the “Intermediate” setting. Find the `ssl_protocols` directive and replace that line with the following three lines: (✔️ Already available in `global/server/ssl.conf`)

```
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305;
ssl_dhparam /etc/nginx/dhparam;
```

This ensures we aren’t allowing the use of old, insecure protocols and ciphers.

Now find the `ssl_prefer_server_ciphers` directive and update it to `off`: (✔️ Already available in `global/server/ssl.conf`)

```
ssl_prefer_server_ciphers off;
```

This allows the client to choose the most performant cipher suite for their hardware configuration from our list of supported ciphers above.

Now let’s download that `dhparam` file that we referenced in the SSL configuration update above and save it to the server: (🔲 Apply after setting up Nginx Kit)

Generate File: (ChatGPT)

```bash
sudo openssl dhparam -out /etc/nginx/dhparam 4096
```

Verify this directive is present: (ChatGPT) (✔️ Already available in `global/server/ssl.conf`)

```nginx
ssl_dhparam /etc/nginx/dhparam;
```

Generate File (Guide)

```
# sudo sh -c 'curl https://ssl-config.mozilla.org/ffdhe2048.txt > /etc/nginx/dhparam'
```



### SSL Performance
HTTPS connections are a lot more resource hungry than regular HTTP connections. This is due to the additional handshake procedure required when establishing a connection. However, it’s possible to cache the SSL session parameters, which will avoid the SSL handshake altogether for subsequent connections. Just remember that security is the name of the game, so you want clients to re-authenticate often. A happy medium of 10 minutes is usually a good starting point. (✔️ Already available in `global/server/ssl.conf`)

```
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off;
```



### OCSP Stapling
OCSP (Online Certificate Status Protocol) stapling is a method used to check the revocation status of your SSL/TLS certificate, which improves both security and performance. By enabling OCSP stapling, your server can cache the OCSP response from the Certificate Authority (CA) and serve it to clients, reducing the need for clients to reach out to the CA directly.

To enable OCSP stapling, add the following directives within the `http` block under the SSL Settings in the nginx.conf: (✔️ Added in `global/server/ssl.conf`)

```
ssl_stapling on;
ssl_stapling_verify on;
```

Now we must enable OCSP stapling for our site. Open our site’s Nginx config file:
```
sudo nano /etc/nginx/sites-available/EXAMPLE.COM
```

Add the following directive under the SSL certificate directives: (✔️ Added in `sites-available/single-site-with-caching.com`)

```
ssl_trusted_certificate /etc/letsencrypt/live/EXAMPLE.COM/chain.pem;
```

These directives enable OCSP stapling and ensure that the OCSP response of your site is verified against the chain of trust using the provided certificate chain file.



### Cross-site Scripting (XSS)
The most effective way to deal with XSS is to ensure that you correctly validate and sanitize all user input in your code, including that within the WordPress admin areas. But most input validation and sanitization is out of your control when you consider third-party themes and plugins. You can however reduce the risk of vulnerability to XSS attacks by configuring Nginx to provide a few additional response headers.

Let’s assume an attacker has managed to embed a malicious JavaScript file into the source code of your site or web application, maybe through a comment form or something similar. By default, the web browser will unknowingly load this external file and allow its contents to execute. Enter the Content Security Policy header, which allows you to define a whitelist of sources that are approved to load assets (JS, CSS, etc.). If the script isn’t on the approved list, it doesn’t get loaded.



#### Content Security Policy (CSP)
Creating a Content Security Policy can require some trial and error, as you need to be careful not to block assets that should be loaded such as those provided by Google or other third party vendors. As such, we’ll define a fairly relaxed policy at the server level and override it on a per-site basis as needed.

Add the following to the Security Headers section inside the http block: (✔️ Available in `global/server/security.conf` as commented. 🔲 Uncomment and test if site breaks)

```
add_header Content-Security-Policy "default-src 'self' https: data: 'unsafe-inline' 'unsafe-eval';" always;
```

This will block any non HTTPS assets from loading.

As an example of an override, you could add the following to the `server` block in your site’s configuration file that will only allow the current domain and a few sources from Google and WordPress.org: (🔲 Apply in per site basis - test throughly if site breaks)

```
sudo nano /etc/nginx/sites-available/EXAMPLE.COM
```
```
add_header Content-Security-Policy "default-src 'self' https://*.google-analytics.com https://*.googleapis.com https://*.gstatic.com https://*.gravatar.com https://*.w.org data: 'unsafe-inline' 'unsafe-eval';" always;
```

You may have noticed that this only deals with external assets, but what about inline scripts? There are two ways you can handle this:

- Completely disable inline scripts by removing `unsafe-inline` and `unsafe-eval` from the Content-Security-Policy. However, this approach can break some third party plugins or themes, so be careful.
- Enable `X-Xss-Protection` which will instruct the browser to filter through user input and ensure suspicious code isn’t output directly to HTML. Although not bulletproof, it’s a relatively simple countermeasure to implement.



#### X-Xss Protection
To enable the `X-Xss-Protection` filter add the following directive below the `Content-Security-Policy` entry: (✔️ Already available in `global/server/security.conf`)

```
add_header X-Xss-Protection "1; mode=block" always;
```



### Clickjacking
Clickjacking is an attack which fools the user into performing an action which they did not intend to, and is commonly achieved through the use of iframes. An article by Troy Hunt has a thorough explanation of clickjacking attacks.

The most effective way to combat this attack vector is to completely disable frame embedding from third party domains. To do this, add the following directive below the `X-Xss-Protection` header: (✔️ Already available in `global/server/security.conf`)

```
add_header X-Frame-Options "SAMEORIGIN" always;
```

This will prevent all external domains from embedding your site directly into their own through the use of the `iframe` tag:

```
<iframe src="http://mydomain.com"</iframe>
```



### MIME Sniffing
MIME sniffing can expose your site to attacks such as “drive-by downloads.” The `X-Content-Type-Options` header counters this threat by ensuring only the MIME type provided by the server is honored. An article by Microsoft explains MIME sniffing in detail.

To disable MIME sniffing add the following directive: (✔️ Already available in `global/server/security.conf`)

```
add_header X-Content-Type-Options "nosniff" always;
```



### Referrer Policy
The `Referrer-Policy` header allows you to control which information is included in the `Referrer` header when navigating from pages on your site. While referrer information can be useful, there are cases where you may not want the full URL passed to the destination server, for example, when navigating away from private content (think membership sites).

In fact, since WordPress 4.9 any requests from the WordPress dashboard will automatically send a blank referrer header to any external destinations. Doing so makes it impossible to track these requests when navigating away from your site (from within the WordPress dashboard), which helps to prevent broadcasting the fact that your site is running on WordPress by not passing `/wp-admin` to external domains.

We can take this a step further by restricting the referrer information for all pages on our site, not just the WordPress dashboard. A common approach is to pass only the domain to the destination server, so instead of:

```
https://myawesomesite.com/top-secret-url
```

```
https://myawesomesite.com
```

You can achieve this using the following policy: (✔️ Already available in `global/server/security.conf` - `"strict-origin-when-cross-origin"`)

```
add_header Referrer-Policy "origin-when-cross-origin" always;
```



### Permissions Policy
The `Permissions-Policy` header allows a site to enable and disable certain browser features and APIs. This allows you to manage which features can be used on your own pages and anything that you embed.

A Permissions Policy works by specifying a directive and an allowlist. The directive is the name of the feature you want to control and the allowlist is a list of origins that are allowed to use the specified feature. MDN has a full list of available directives and allowlist values. Each directive has its own default allowlist, which will be the default behavior if they are not explicitly listed in a policy.

You can specify several features at the same time by using a comma-separated list of policies. In the following example, we allow geolocation across all contexts, we restrict the camera to the current page and the specified domain, and we block the microphone across all contexts: 

(✔️ Already available in `global/server/security.conf` - `"geolocation=(), microphone=(), camera=()" always;`)

```
add_header Permissions-Policy "geolocation=*, camera=(self 'https://example.com'), microphone=()";
```

