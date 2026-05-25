---
title: Installation
description: Guide showing you how to install the panel
order: 1
type: guide
---

Vessel is designed to run on your own web server. You will need root access to your machine in order to install and manage the panel.

## Requirements

- Node.js `v18` or higher
- A web server (Nginx, Apache, or Caddy)
- A domain or subdomain pointed at your server

## Installing Node.js

If you don't already have Node.js installed, the easiest way is via [nvm](https://github.com/nvm-sh/nvm).

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 18
nvm use 18
```

Verify the installation:

```bash
node -v
npm -v
```

## Downloading Vessel

Clone the repository and navigate into the panel directory.

```bash
git clone https://github.com/vessel-db/panel.git /var/www/vessel
cd var/www/vessel
```

## Installing Dependencies

```bash
npm install
```

## Building the Panel

```bash
npm run build
```

## Running the Panel

Start the panel server:

```bash
node dist/server/entry.mjs
```

The panel will start on port `4321` by default. It is recommended to run this as a background process using a process manager like PM2.

### Using PM2

Install PM2 globally:

```bash
npm install -g pm2
```

Start the panel with PM2:

```bash
pm2 start dist/server/entry.mjs --name vessel-panel
pm2 save
pm2 startup
```

The panel will now automatically restart on reboot.

## Next Steps

With the panel running, you need to configure a web server to reverse proxy requests to it. See the [Web Server Configuration](/docs/panel/web-server-configuration) guide.