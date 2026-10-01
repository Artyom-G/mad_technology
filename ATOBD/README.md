# VM 101: atobd

Debian 13 VM running NAS (Samba) + Jellyfin.

## Proxmox

- VMID: `101`
- Name: `atobd`
- LAN IP: `192.168.68.12`
- Tailscale IP: `100.83.234.7` (hostname `atobd`)
- Proxmox host: `192.168.68.10`

Web UI: `https://192.168.68.10:8006`

From the Proxmox shell:

```
qm status 101
qm start 101
qm shutdown 101
qm reboot 101
qm config 101
```

Starts automatically with the host (`onboot: 1`, `startup: order=1`).

## VM config (known-good baseline)

- BIOS: SeaBIOS, machine `q35`
- CPU: 2 cores, `x86-64-v2-AES`
- RAM: 4 GB
- OS disk: 32 GB on `local-lvm` (VirtIO SCSI single)
- NIC: VirtIO, MAC `BC:24:11:0C:4C:C2`, bridge `vmbr0`
- USB passthrough: `usb0: host=152d:a578` (4 TB drive)
- No GPU passthrough, no OVMF, no ISO attached

## SSH

On the LAN:

```
ssh melon@192.168.68.12
```

Remotely (over Tailscale):

```
ssh melon@atobd
```

To reach other LAN machines remotely, jump through this VM:

```
ssh -J melon@atobd root@192.168.68.10
```

## Jellyfin

- LAN: `http://192.168.68.12:8096`
- Remote: `http://atobd:8096` (Tailscale)
- Libraries: `/mnt/storage/movies`, `/mnt/storage/tv`
- Transcoding is CPU-only (no GPU), so prefer direct play.

## SMB / NAS

From Windows: `\\192.168.68.12`
Remotely: `\\100.83.234.7`

## Storage

The 4 TB HDD (`ST4000DM004`, in a SABRENT/JMicron USB enclosure, `152d:a578`) is passed through from the Proxmox host over USB.

Mounted in the guest at `/mnt/storage`.

Directories: `backups`, `files`, `movies`, `music`, `photos`, `tv`

fstab entry:

```
UUID=2ec6d966-a5b6-472e-b2da-a87ca7571c5e /mnt/storage ext4 defaults,nofail,x-systemd.device-timeout=10 0 2
```

`nofail` lets the VM boot even if the drive is missing. Without it, a missing drive drops the guest into emergency mode with no networking. Check the mount with `findmnt /mnt/storage`. If the drive is missing, Samba and Jellyfin will see an empty folder.

Before detaching the drive, shut the VM down cleanly.

## Network

- Gateway: `192.168.68.1`
- Guest interface: `ens18`, static `192.168.68.12/24`
- Remote access: Tailscale installed in this VM (key expiry disabled)

## Troubleshooting

If the VM is running but unreachable:

1. Take a screenshot of the console: `qm monitor 101`, then `screendump /tmp/vm101.ppm`. `qm terminal` is blank, because the guest isn't configured for serial output.
2. If it shows emergency mode, check that the 4 TB drive is visible with `lsusb | grep 152d` and attached in `qm config 101`.

## GPU passthrough (shelved)

An attempt to pass the Vega 11 iGPU for Jellyfin transcoding was abandoned. The guest `amdgpu` driver reported no usable ROM (`Unable to locate a BIOS ROM`), and the host logged IOMMU timeouts. Leftover host settings, which are harmless while no VM uses `hostpci0`:

- `/etc/modprobe.d/vfio.conf`: `options vfio-pci ids=1002:15d8`
- GRUB: `quiet iommu=pt pci=noats`
