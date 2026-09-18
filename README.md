# Mad Technology Homelab

Documentation and configuration notes for my self-hosted homelab.

## Structure

```text
homelab/
├── README.md
├── brothers-grim/
│   └── README.md
```

### Directories

* **`./`** — Proxmox host hardware, networking, and VM configuration.
* **`brothers-grim/`** — Main Web Server VM (Debian) and its hosted applications.
* **`ATOBD/`** — NAS storage (Debian + Samba + Jellyfin), shares, and media server.

Each VM and major service has its own `README.md` containing its configuration, purpose, and useful commands.
