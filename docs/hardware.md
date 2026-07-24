# Hardware

## Current Host

| Component | Spec |
|---|---|
| Model | HP ProDesk 600 G1 |
| CPU | Intel Core i5-4570 (4 cores / 4 threads, no Hyper-Threading, Haswell) |
| RAM | 8 GB DDR3 |
| Storage | Two drives — see below, **no redundancy on either** |
| Hypervisor | Proxmox VE, freshly reinstalled onto the new SSD  |
| iGPU | Intel HD Graphics 4600 (built into the i5-4570) |

| Drive | Role |
|---|---|
| New 500 GB SSD | Boot drive — fresh Proxmox VE install |
| Original HDD | Not yet assigned in Proxmox |

## Storage note



**Next step:** assign the HDD in Proxmox as its own storage, dedicated to bulk/sequential data —
Jellyfin's media library and a future backup target — rather than general VM/CT disks. Spinning-
disk latency is a poor fit for anything database-like (Docker volumes, Minecraft world saves), so
those stay on the boot SSD's storage pool, capacity permitting. Document once done: how the HDD
was added (directory storage vs. LVM vs. ZFS single-disk) and what got pointed at it.

## Hardware transcoding (Jellyfin)

The i5-4570's HD Graphics 4600 iGPU supports Intel Quick Sync Video, which Jellyfin can use for
hardware-accelerated transcoding instead of falling back to CPU (software) transcoding. To use
it from inside a Proxmox VM or LXC, the iGPU needs to be passed through:
- **LXC (unprivileged container):** bind-mount `/dev/dri` into the container — simplest path,
  no full GPU passthrough needed.
- **VM:** requires full PCI passthrough of the iGPU to the VM, which is more involved and means
  the Proxmox host itself loses console output via that GPU.

Jellyfin needs to be reinstalled after the fresh Proxmox install — when rebuilding it, bind-mount
`/dev/dri` into the LXC from the start rather than adding it as a follow-up fix later.

## Future hardware considerations

- **RAM/CPU headroom:** the i5-4570 has no Hyper-Threading, so heavier concurrent workloads
  (e.g. running Docker + Pi-hole + a VPN endpoint + Grafana/Prometheus + a Minecraft server all
  at once) are worth watching for CPU contention, even though each service individually is light.
- **Additional drives:** the HDD still needs to be assigned in Proxmox (see the storage note
  above); a NAS build later will need further drives beyond that — see the ZFS note in
  `future-plans.md`.
