# Proxmox

## Overview

The homelab runs on a Lenovo ThinkCentre M75s-1 SFF running Proxmox VE.

Proxmox provides the virtualization layer for the homelab and hosts the various VMs used for websites, services, and future infrastructure.

## Hardware

| Component | Specification                 |
| --------- | ----------------------------- |
| System    | Lenovo ThinkCentre M75s-1 SFF |
| CPU       | AMD Ryzen 5 PRO 3400G         |
| CPU       | 4 cores / 8 threads           |
| RAM       | 16 GB DDR4                    |
| Storage   | 256 GB SATA SSD               |
| GPU       | AMD Vega 11                   |
| Network   | Gigabit Ethernet              |
| OS        | Proxmox VE 9.2.2              |
| Kernel    | 7.0.2-6-pve                   |

## Network

The Proxmox host is connected to the home LAN through Ethernet.

| Setting        | Value             |
| -------------- | ----------------- |
| LAN            | `192.168.68.0/24` |
| Gateway        | `192.168.68.1`    |
| Proxmox bridge | `vmbr0`           |
| Proxmox IP     | `192.168.68.10`   |

The Proxmox host previously had a USB Wi-Fi adapter configured as a fallback, but Ethernet is the primary network connection.

## Virtual Machines

### Debian Server

| Setting           | Value                     |
| ----------------- | ------------------------- |
| Purpose           | Web server / self-hosting |
| IP                | `192.168.68.11`           |
| OS                | Debian 13                 |
| Network interface | `ens18`                   |

See [`../debian-server/README.md`](../debian-server/README.md).


## Proxmox Configuration

### Repository

The system uses the Proxmox no-subscription repository for a non-production homelab installation.

The enterprise repository should remain disabled because this system does not have a Proxmox subscription.

## Networking

The primary bridge is:

```text
vmbr0
  │
  └── Physical Ethernet
       │
       └── Home Router
```

VMs are connected to `vmbr0` and receive addresses on the `192.168.68.0/24` LAN.

## Management

Proxmox is managed through its web interface and SSH.

The host should remain accessible from the local network even if individual VMs or services are unavailable.

## Notes

* Proxmox is the virtualization layer; applications should generally run inside VMs rather than directly on the Proxmox host.
* Avoid installing unnecessary applications directly on the Proxmox host.
* VM networking uses the existing home LAN.
* The NAS will eventually use a separate VM.
