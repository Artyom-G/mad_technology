# Debian Server

## Overview

The Debian Server VM is the primary self-hosting/web server in the homelab.

It hosts static websites and provides the local origin for services exposed through Cloudflare Tunnel.

## System

| Setting   | Value           |
| --------- | --------------- |
| OS        | Debian 13       |
| Host      | Proxmox         |
| IP        | `192.168.68.11` |
| Interface | `ens18`         |
| Gateway   | `192.168.68.1`  |
| DNS       | `8.8.8.8`       |

## Network

The VM uses a static IPv4 address.

```text
IP:       192.168.68.11/24
Gateway:  192.168.68.1
DNS:      8.8.8.8
```

The VM is connected to the Proxmox `vmbr0` bridge.

## Services

### Caddy

Caddy is the web server and local HTTP origin.

It serves the static websites hosted on this VM.

Current Caddy configuration:

```text
http://test.artyomg.com {
    root * /home/melon/projects/artyomg.com_next/frontend/out
    file_server
}

http://artyomg.com {
    root * /home/melon/projects/artyomg.com_next/frontend/out
    file_server
}
```

Caddy listens for HTTP traffic locally.

HTTPS is handled externally by Cloudflare.

### Cloudflare Tunnel

`cloudflared` runs as a systemd service.

The tunnel provides public access to services without requiring inbound port forwarding from the home router.

Current public routes include:

```text
test.artyomg.com → http://localhost:80
artyomg.com      → http://localhost:80
```

Cloudflare terminates HTTPS before traffic is sent through the tunnel to the Debian server.

## Websites

### artyomg.com

Source/project:

```text
/home/melon/projects/artyomg.com_next/
```

Frontend build output:

```text
/home/melon/projects/artyomg.com_next/frontend/out/
```

The site is statically exported and served by Caddy.

### test.artyomg.com

Currently points to the same static build directory as `artyomg.com`:

```text
/home/melon/projects/artyomg.com_next/frontend/out/
```

This subdomain is used for testing the deployment and Cloudflare configuration.

## Project Management

Projects are kept under:

```text
/home/melon/projects/
```

This allows the projects to be managed directly with Git.

Typical deployment workflow:

```bash
cd /home/melon/projects/<project>
git pull
npm install
npm run build
```

For a Next.js static export, the resulting site is placed in:

```text
out/
```

## Caddy Permissions

The Caddy service runs under the `caddy` user.

Because the websites are stored under `/home/melon`, Caddy requires permission to traverse the parent directories and read the generated website files.

ACLs are used to grant Caddy the required access without changing ownership of the projects.

## Cloudflare Architecture

The current architecture is:

```text
Internet
    │
    ▼
Cloudflare
    │
    │ HTTPS
    ▼
Cloudflare Tunnel
    │
    │ outbound connection
    ▼
Debian Server
    │
    ▼
Caddy :80
    │
    ▼
Static website
```

The tunnel is initiated from the Debian server, so the server does not need to accept inbound connections from the Internet for these websites.

## Security

Public HTTPS is provided by Cloudflare.

The local Caddy origin uses HTTP because the connection between Cloudflare Tunnel and Caddy remains inside the server.

When Cloudflare Tunnel is being used exclusively, unnecessary router port forwards for HTTP/HTTPS should be removed.

## Useful Commands

### Check Caddy

```bash
sudo systemctl status caddy
```

Reload configuration:

```bash
sudo systemctl reload caddy
```

Validate configuration:

```bash
sudo caddy validate --config /etc/caddy/Caddyfile
```

### Check Cloudflare Tunnel

```bash
sudo systemctl status cloudflared
```

View logs:

```bash
sudo journalctl -u cloudflared -f
```

### Check network

```bash
ip addr
```

```bash
ip route
```

Test connectivity:

```bash
ping 8.8.8.8
```

Test DNS:

```bash
ping google.com
```

## Future Services

This VM may eventually host additional web-facing applications.

Storage-heavy services such as the NAS and media server should be kept on a separate VM rather than being added to this server.

## Notes

* Keep source code under `/home/melon/projects/`.
* Do not manually edit generated frontend output.
* Rebuild the frontend after pulling source changes.
* Caddy serves generated static files.
* Cloudflare Tunnel handles public access.
* The NAS/media server will be deployed separately.
