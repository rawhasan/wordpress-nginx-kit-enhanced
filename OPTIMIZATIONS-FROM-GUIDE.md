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

Now let’s update our SSL configuration as per the recommendations of Mozilla’s SSL Configuration Generator on the “Intermediate” setting. Find the `ssl_protocols` directive and replace that line with the following three lines:
```
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305;
ssl_dhparam /etc/nginx/dhparam;
```

This ensures we aren’t allowing the use of old, insecure protocols and ciphers.

Now find the `ssl_prefer_server_ciphers` directive and update it to `off`:
```
ssl_prefer_server_ciphers off;
```

This allows the client to choose the most performant cipher suite for their hardware configuration from our list of supported ciphers above.

Now let’s download that `dhparam` file that we referenced in the SSL configuration update above and save it to the server:
```
sudo sh -c 'curl https://ssl-config.mozilla.org/ffdhe2048.txt > /etc/nginx/dhparam'
```



