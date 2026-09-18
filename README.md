# Mad Technology Homelab

Documentation and configuration notes for my self-hosted homelab.

## Structure

```text
homelab/
├── README.md
├── proxmox/
│   └── README.md
├── debian-server/
│   └── README.md
├── nas/
│   └── README.md
├── networking/
│   └── README.md
└── services/
    ├── caddy/
    │   └── README.md
    ├── cloudflare/
    │   └── README.md
    └── ...
```

### Directories

* **`proxmox/`** — Proxmox host hardware, networking, and VM configuration.
* **`debian-server/`** — Main Debian VM and its hosted applications.
* **`nas/`** — NAS storage, shares, and media server.
* **`networking/`** — Network topology, IPs, and router configuration.
* **`services/`** — Documentation for individual services.

Each VM and major service has its own `README.md` containing its configuration, purpose, and useful commands.
