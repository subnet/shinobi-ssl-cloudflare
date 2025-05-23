# 🔒 Secure Shinobi with Let's Encrypt & Cloudflare DNS

This guide walks you through securing your **Shinobi CCTV system** with a **free SSL certificate** from **Let’s Encrypt**, using **Certbot** with **Cloudflare DNS API Token validation**. Perfect for LAN-only or internal deployments!

---

## 🚀 Why Use Cloudflare DNS Validation?

 ✅ No need to open ports on your firewall/router  
 ✅ Works even with non-public, local-only domains  
 ✅ Fully automated certificate issuance via Cloudflare API  

> 💡 You must already have a domain on Cloudflare and an API token created with DNS edit permissions.

---

## 🛠 Step-by-Step Guide

> All commands must be run as `root` or using `sudo`.

### 1. Update the System
```bash
apt-get update && apt-get dist-upgrade
```

### 2. Install Certbot and DNS Plugin
```bash
apt install -y certbot python3-certbot-dns-cloudflare
```

### 3. Prepare Cloudflare Credentials
```bash
mkdir -p /etc/cloudflare
chmod 700 /etc/cloudflare
touch /etc/cloudflare/shinobi.yourdomain.com.ini
chmod 600 /etc/cloudflare/shinobi.yourdomain.com.ini
```

Save your Cloudflare API token:
```bash
echo "dns_cloudflare_api_token = YOUR_CLOUDFLARE_API_TOKEN" > /etc/cloudflare/shinobi.yourdomain.com.ini
```

Replace \`YOUR_CLOUDFLARE_API_TOKEN\` with your actual token.

### 4. Obtain the SSL Certificate
```bash
certbot certonly --register-unsafely-without-email \
  --dns-cloudflare \
  --dns-cloudflare-credentials /etc/cloudflare/shinobi.yourdomain.com.ini \
  -d shinobi.yourdomain.com \
  --dns-cloudflare-propagation-seconds 60
```

### 5. Configure Shinobi

#### Stop Shinobi:
```bash
pm2 stop /opt/Shinobi/camera.js && pm2 stop /opt/Shinobi/cron.js
```

#### Edit Configuration:
```bash
cd /opt/Shinobi/
nano conf.json
```

Update (or add) the "ssl" section:
```json
"ssl": {
  "key":"/etc/letsencrypt/live/shinobi.yourdomain.com/privkey.pem",
  "cert":"/etc/letsencrypt/live/shinobi.yourdomain.com/fullchain.pem",
  "port": 443
}
```

Save and exit.

#### Restart Shinobi:
```bash
pm2 start /opt/Shinobi/camera.js && pm2 start /opt/Shinobi/cron.js
```

---

## ✅ Final Check

Visit your Shinobi instance in the browser:

```
https://shinobi.yourdomain.com
```

You should now see your dashboard secured with HTTPS. 🎉

---

## 🧩 Additional Tips

- Certbot should renew the certificate automatically, but you can set a cron if you want
  ```bash
  0 3 * * * certbot renew --quiet && pm2 restart all
  ```
- Use a reverse proxy like Nginx if needed for advanced TLS settings.
- Keep your Cloudflare API token secure and with minimal permissions.

---

## 📄 License

This guide is provided under the WTFPL License (https://www.wtfpl.net/)

---

## 🙌 Credits

Created by [@subnet]  
Inspired by the need to secure CCTV systems with minimal exposure.
