---
title: Web Server Configuration
description: Showing how to configure the web server for the panel
order: 2
type: guide
---

# Web Server Configuration

The Vessel panel runs on port `4321` and needs to be reverse proxied through a web server. Pick the web server you are using below.

## Nginx

Install Nginx if you haven't already:

```bash
apt install -y nginx
```

Create a new site configuration:

```bash
nano /etc/nginx/sites-available/vessel.conf
```

Paste the following, replacing `panel.example.com` with your domain:

```nginx
server {
    listen 80;
    server_name panel.example.com;

    location / {
        proxy_pass http://localhost:4321;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable the site and reload Nginx:

```bash
ln -s /etc/nginx/sites-available/vessel.conf /etc/nginx/sites-enabled/vessel.conf
nginx -t
systemctl reload nginx
```

---

## Apache

Install Apache if you haven't already:

```bash
apt install -y apache2
```

Enable the required modules:

```bash
a2enmod proxy proxy_http proxy_wstunnel rewrite headers
```

Create a new virtual host configuration:

```bash
nano /etc/apache2/sites-available/vessel.conf
```

Paste the following, replacing `panel.example.com` with your domain:

```apache
<VirtualHost *:80>
    ServerName panel.example.com

    ProxyPreserveHost On
    ProxyPass / http://localhost:4321/
    ProxyPassReverse / http://localhost:4321/

    RequestHeader set X-Forwarded-Proto "http"
    RequestHeader set X-Real-IP "%{REMOTE_ADDR}s"

    ErrorLog ${APACHE_LOG_DIR}/vessel_error.log
    CustomLog ${APACHE_LOG_DIR}/vessel_access.log combined
</VirtualHost>
```

Enable the site and reload Apache:

```bash
a2ensite vessel.conf
apache2ctl configtest
systemctl reload apache2
```

---

## Caddy

Install Caddy if you haven't already. See [caddyserver.com/docs/install](https://caddyserver.com/docs/install) for instructions.

Open your `Caddyfile`:

```bash
nano /etc/caddy/Caddyfile
```

Add the following, replacing `panel.example.com` with your domain:

```caddy
panel.example.com {
    reverse_proxy localhost:4321
}
```

Reload Caddy:

```bash
systemctl reload caddy
```

Caddy automatically provisions and renews SSL certificates via Let's Encrypt.

---

## Cloudflare Tunnel

Cloudflare Tunnels let you expose the panel without opening any ports or configuring SSL manually. Cloudflare handles HTTPS automatically.

Install `cloudflared`:

```bash
curl -L https://pkg.cloudflare.com/cloudflare-main.gpg | tee /usr/share/keyrings/cloudflare-main.gpg > /dev/null
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared any main' | tee /etc/apt/sources.list.d/cloudflared.list
apt update && apt install -y cloudflared
```

Authenticate with your Cloudflare account:

```bash
cloudflared tunnel login
```

Create a tunnel:

```bash
cloudflared tunnel create vessel
```

Create a config file at `~/.cloudflared/config.yml`, replacing `panel.example.com` with your domain and `<TUNNEL_ID>` with the ID output from the previous command:

```yaml
tunnel: <TUNNEL_ID>
credentials-file: /root/.cloudflared/<TUNNEL_ID>.json

ingress:
  - hostname: panel.example.com
    service: http://localhost:4321
  - service: http_status:404
```

Route your domain to the tunnel:

```bash
cloudflared tunnel route dns vessel panel.example.com
```

Start the tunnel as a service:

```bash
cloudflared service install
systemctl enable --now cloudflared
```

No web server or SSL configuration is needed when using Cloudflare Tunnels.

---

## SSL (Nginx & Apache)

If you are using Nginx or Apache, it is strongly recommended to enable HTTPS using Certbot.

Install Certbot:

```bash
apt install -y certbot python3-certbot-nginx
# or for Apache:
apt install -y certbot python3-certbot-apache
```

Run Certbot:

```bash
# Nginx
certbot --nginx -d panel.example.com

# Apache
certbot --apache -d panel.example.com
```

Certbot will automatically configure SSL and set up auto-renewal.