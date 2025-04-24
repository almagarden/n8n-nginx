# Self-Hosted n8n with Nginx Reverse Proxy

This guide helps you self-host `n8n`, served at `https://n8n.<your-domain>` and a static website (e.g. `index.html`) at `https://<your-domain>`, using Nginx as a reverse proxy with SSL certificates provided by Cloudflare.

## Prerequisites

- A Linux server with Docker and Docker Compose installed.
- A domain name managed via Cloudflare.
- Two separate Docker Compose projects: one for `n8n`, one for Nginx.

---

## 🛠️ Setup Steps

### 1. Add SSL Certificates

1. In the [Cloudflare dashboard](https://dash.cloudflare.com):
   - Go to **SSL/TLS > Overview**
   - Set SSL mode to **Full (Strict)**
   - Go to **Origin Server** > **Create Certificate**
2. Save the certificate and key files (in `.pem` format).
3. Place them into your Nginx certs directory:
   ```bash
   cp origin-cert.pem nginx-proxy/certs/
   cp origin-key.pem nginx-proxy/certs/
   ```

---

### 2. Configure DNS Records in Cloudflare

In **Cloudflare > DNS > Records**, add the following **A** records:

| Type | Name        | IPv4 Address       |
|------|-------------|--------------------|
| A    | @           | `<your_server_ip>` |
| A    | n8n         | `<your_server_ip>` |

---

### 3. Configure n8n Environment

Edit the `n8n-compose/.env` file:

```env
DOMAIN_NAME=your-domain.com
SUBDOMAIN=n8n
```

---

### 4. Configure Nginx Reverse Proxy

Edit `nginx-proxy/nginx/default.conf`, and replace placeholders:

```nginx
server {
    listen 80;
    server_name your-domain.com n8n.your-domain.com;

    return 301 https://$host$request_uri;
}

server {
    server_name your-domain.com;
    ssl_certificate /etc/nginx/certs/origin-cert.pem;
    ssl_certificate_key /etc/nginx/certs/origin-key.pem;
    ...
}

server {
    server_name n8n.your-domain.com;
    ssl_certificate /etc/nginx/certs/origin-cert.pem;
    ssl_certificate_key /etc/nginx/certs/origin-key.pem;
    ...
}
```

---

### 5. Launch Services

Start your containers:

```bash
docker compose up -d
```

---

✅ Done! Your services should now be live at:
- `https://<your-domain>` for your website
- `https://n8n.<your-domain>` for the n8n instance
