# Measuring the Effect of OCSP Stapling

OCSP stapling can improve both performance and security. This guide shows how to test and quantify its impact.

---

## ✅ 1. What Does OCSP Stapling Improve?

### Without Stapling:
- Browsers contact the Certificate Authority (CA) directly to check certificate revocation
- Adds ~300–500 ms to TLS handshake
- Fails if the OCSP server is slow or unreachable
- Leaks privacy (CA sees which sites you're visiting)

### With Stapling:
- The server provides the OCSP response
- Faster handshake (no external OCSP fetch)
- Better privacy and reliability
- Helps maintain **Perfect Forward Secrecy**

---

## ✅ 2. How to Verify OCSP Stapling Is Working

### Using `openssl`:

```bash
openssl s_client -connect yourdomain.com:443 -status
```

Look for output like:

```
OCSP Response Status: successful (0x0)
Cert Status: good
```

✅ This confirms that the server is stapling OCSP responses.

---

## ✅ 3. Measure TLS Handshake Time with `curl`

Create a custom format file:

### `curl-format.txt`

```text
\n
DNS Lookup:        %{time_namelookup}s
Connect:           %{time_connect}s
TLS Handshake:     %{time_appconnect}s
Total Time:        %{time_total}s
\n
```

### Then run:

```bash
curl -w "@curl-format.txt" -o /dev/null -s https://yourdomain.com
```

Compare results **with and without** OCSP stapling in Nginx.

---

## ✅ 4. Check in Browser (Chrome / Firefox)

### Chrome DevTools:
- Open Chrome
- Visit `https://yourdomain.com`
- Open DevTools → **Security** tab
- Look for: `Certificate is valid (OCSP stapled)`

### Firefox:
- Go to `about:networking#security`
- Find your site → check OCSP status

---

## ✅ 5. Use SSL Labs

Visit:
- [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/)

Look for:
- **OCSP Stapling: Yes**
- Check handshake simulation for major browsers

---

## ✅ 6. Monitor with Web Tools

You can also benchmark performance before and after enabling OCSP stapling:

- [WebPageTest](https://www.webpagetest.org/) → Time to First Byte (TTFB)
- Google Lighthouse (`Ctrl+Shift+I → Lighthouse`) → Network timing
- `curl` or `openssl` in CI/CD health checks
- `prometheus + blackbox_exporter` for automated TLS checks

---

## 📊 Summary: OCSP Stapling Impact

| Category             | With Stapling ✅      | Without Stapling ❌   |
|----------------------|-----------------------|------------------------|
| TLS Handshake Time   | ✅ Faster              | ❌ Slower              |
| Privacy              | ✅ Encrypted, private  | ❌ CA sees every check |
| Revocation Check     | ✅ Offline/resilient   | ❌ Dependent on CA     |
| SSL Labs Score       | ✅ Improved            | ⚠ May show warning     |
| Browser Compatibility| ✅ Preferred           | ⚠ Might fallback       |

---

## ✅ Recommendation

Always enable OCSP stapling **if supported by your certificate authority** and your server stack. It improves both **security** and **performance** with minimal configuration.

