# Enabling OCSP Stapling with Nginx and acme.sh

Certbot was not providing a certificate with OCSP Support. So issued a certificate with `acme.sh` and took the steps to make the OCSP work.

This guide explains how to issue an SSL certificate that supports OCSP stapling using `acme.sh`, configure it in Nginx, and verify it works.

---

## ✅ Prerequisites

- A server running Nginx
- Shell access with `sudo` privileges
- Domain is already pointed to the server

---

## 🧩 Step 1: Install `acme.sh`

```bash
cd ~
curl https://get.acme.sh | sh
source ~/.acme.sh/acme.sh.env
```

Optional (add to PATH permanently):

```bash
echo 'export PATH="$HOME/.acme.sh:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 🧩 Step 2: Switch to Root (required for webroot write access)

```bash
sudo -i
```

---

## 🧩 Step 3: Register Account with Email

```bash
/home/youruser/.acme.sh/acme.sh --register-account -m your@email.com
```

---

## 🧩 Step 4: Issue Certificate with OCSP + ISRG Chain

Use your actual webroot path (e.g., `/sites/yourdomain.com/public`):

```bash
/home/youruser/.acme.sh/acme.sh --issue \
  -d yourdomain.com \
  --webroot /sites/yourdomain.com/public \
  --preferred-chain "ISRG Root X1" \
  --ocsp
```

---

## 🧩 Step 5: Copy Cert Files to Nginx Directory

```bash
mkdir -p /etc/nginx/ssl/yourdomain.com

cp /root/.acme.sh/yourdomain.com_ecc/yourdomain.com.cer /etc/nginx/ssl/yourdomain.com/cert.pem
cp /root/.acme.sh/yourdomain.com_ecc/yourdomain.com.key /etc/nginx/ssl/yourdomain.com/privkey.pem
cp /root/.acme.sh/yourdomain.com_ecc/ca.cer /etc/nginx/ssl/yourdomain.com/chain.pem
```

---

## 🧩 Step 6: Update Nginx SSL Configuration

In your Nginx server block:

```nginx
ssl_certificate     /etc/nginx/ssl/yourdomain.com/cert.pem;
ssl_certificate_key /etc/nginx/ssl/yourdomain.com/privkey.pem;
ssl_trusted_certificate /etc/nginx/ssl/yourdomain.com/chain.pem;

ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 1.0.0.1 valid=300s;
resolver_timeout 5s;
```

---

## 🧩 Step 7: Reload Nginx

```bash
nginx -t && systemctl reload nginx
```

---

## 🧪 Step 8: Verify OCSP Stapling

```bash
openssl s_client -connect yourdomain.com:443 -status
```

You should see:

```
OCSP Response Status: successful (0x0)
Cert Status: good
```

---

## 🔄 Optional: Automate Renewal

Edit root crontab:

```bash
sudo crontab -e
```

Add:

```bash
0 2 * * * /root/.acme.sh/acme.sh --cron --home /root/.acme.sh > /var/log/acme-renew.log 2>&1 && systemctl reload nginx
```

---

## ✅ Done!

You now have OCSP stapling correctly configured using `acme.sh` and Nginx.








# 🔁 Switching `acme.sh` Back to Let's Encrypt (with OCSP Stapling)

If you're currently using ZeroSSL with `acme.sh` and want to switch to Let's Encrypt (e.g., to stay within an open-source CA or to align with OCSP support), follow this guide.

---

## ✅ Step 1: Set Let's Encrypt as the Default CA

Run this as your **non-root user** (e.g. `btaworld`):

```bash
acme.sh --set-default-ca --server letsencrypt
```

You should see output like:

```
Changed default CA to: https://acme-v02.api.letsencrypt.org/directory
```

---

## ✅ Step 2: Re-Issue the Certificate with Let's Encrypt

Switch to root or a user with write access to your webroot:

```bash
sudo -i
```

Then run:

```bash
/home/youruser/.acme.sh/acme.sh --issue \
  -d yourdomain.com \
  --webroot /path/to/your/webroot \
  --preferred-chain "ISRG Root X1" \
  --ocsp
```

**Replace:**
- `youruser` with the actual user (`btaworld`, etc.)
- `yourdomain.com` with your domain
- `/path/to/your/webroot` with your real document root (e.g. `/var/www/html` or `/sites/yourdomain.com/public`)

---

## ✅ Step 3: Copy the New Certificate to Nginx SSL Directory

```bash
mkdir -p /etc/nginx/ssl/yourdomain.com

cp /root/.acme.sh/yourdomain.com_ecc/yourdomain.com.cer /etc/nginx/ssl/yourdomain.com/cert.pem
cp /root/.acme.sh/yourdomain.com_ecc/yourdomain.com.key /etc/nginx/ssl/yourdomain.com/privkey.pem
cp /root/.acme.sh/yourdomain.com_ecc/ca.cer /etc/nginx/ssl/yourdomain.com/chain.pem
```

---

## ✅ Step 4: Configure Nginx for OCSP Stapling

In your SSL server block:

```nginx
ssl_certificate     /etc/nginx/ssl/yourdomain.com/cert.pem;
ssl_certificate_key /etc/nginx/ssl/yourdomain.com/privkey.pem;
ssl_trusted_certificate /etc/nginx/ssl/yourdomain.com/chain.pem;

ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 1.0.0.1 valid=300s;
resolver_timeout 5s;
```

---

## ✅ Step 5: Reload Nginx

```bash
nginx -t && systemctl reload nginx
```

---

## ✅ Step 6: Verify OCSP Stapling

Use `openssl` to confirm OCSP stapling is now active:

```bash
openssl s_client -connect yourdomain.com:443 -status
```

You should see:

```
OCSP Response Status: successful (0x0)
Cert Status: good
```

---

## 🔁 Optional: Automate Renewal and Reload

Add this cron job to root:

```bash
sudo crontab -e
```

Then add:

```bash
0 2 * * * /root/.acme.sh/acme.sh --cron --home /root/.acme.sh > /var/log/acme-renew.log 2>&1 && systemctl reload nginx
```

---

## ✅ Done!

You're now using **Let’s Encrypt** again with **OCSP stapling** correctly configured in Nginx.

