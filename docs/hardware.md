# Hardware

## Current Host

| Component | Spec |
|---|---|
| Model | HP ProDesk 600 G1 |
| CPU | Intel Core i5-4570 (4 cores / 4 threads, no Hyper-Threading, Haswell) |
| RAM | 8 GB DDR3 |
| Storage | Single 500 GB SSD — only drive currently in the machine, **no redundancy**, see note below |
| Hypervisor | Proxmox VE, latest release at time of install (exact version not pinned — run `pveversion` on the host to record it precisely) |
| iGPU | Intel HD Graphics 4600 (built into the i5-4570) |

Note: the day-01 journal entry originally described the drive as an HDD; it's actually a 500 GB
SSD — updated here to match reality.

## Storage note

This is currently a **single point of failure** — one 500 GB SSD, no RAID, no redundancy, and
it's serving as both the Proxmox boot drive and where all VM/CT data lives. That's fine for a
learning setup, but it's worth keeping in mind before anything stateful (Minecraft world data,
Jellyfin watch history/config, future backups) depends on this disk. See the storage sequencing
note in [`future-plans.md`](./future-plans.md) — backups are recommended before adding more
stateful services, and a NAS build later is the longer-term fix for redundancy.

**Planned this week:** adding a second drive and reconfiguring so the current 500 GB SSD becomes
a dedicated boot drive, with the new drive taking over VM/CT storage. This is a good move — it
separates the OS/hypervisor from workload data, so a boot-drive failure doesn't necessarily take
VM disks down with it (and vice versa). It doesn't add redundancy by itself (still one drive per
role, no mirroring), but it's a meaningful step toward the eventual NAS/ZFS setup, and worth
documenting once done: drive model, size, and how storage was assigned in Proxmox (e.g. new LVM/
ZFS pool for VM disks vs. the boot drive's own partition).

## Hardware transcoding (Jellyfin)

The i5-4570's HD Graphics 4600 iGPU supports Intel Quick Sync Video, which Jellyfin can use for
hardware-accelerated transcoding instead of falling back to CPU (software) transcoding. To use
it from inside a Proxmox VM or LXC, the iGPU needs to be passed through:
- **LXC (unprivileged container):** bind-mount `/dev/dri` into the container — simplest path,
  no full GPU passthrough needed.
- **VM:** requires full PCI passthrough of the iGPU to the VM, which is more involved and means
  the Proxmox host itself loses console output via that GPU.

Given Jellyfin is already installed, worth checking whether `/dev/dri` exists on the host and
is currently exposed to wherever Jellyfin is running — if not, this is a concrete follow-up for
the "finish configuring Jellyfin" item on the roadmap.

## Future hardware considerations

- **RAM/CPU headroom:** the i5-4570 has no Hyper-Threading, so heavier concurrent workloads
  (e.g. running Docker + Pi-hole + a VPN endpoint + Grafana/Prometheus + a Minecraft server all
  at once) are worth watching for CPU contention, even though each service individually is light.
- **Additional drives:** a second drive is planned this week (see the storage note above); a NAS
  build later will need further drives beyond that — see the ZFS note in `future-plans.md`.
