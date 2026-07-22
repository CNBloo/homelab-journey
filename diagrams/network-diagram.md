# Network Diagram

**Status: proposed, not confirmed.** No current subnet/router/DHCP details are documented
anywhere in this repo yet (see [`../docs/networking.md`](../docs/networking.md)). This diagram
is a *starting-point recommendation* to compare against the actual home network setup and adjust
— it is not a record of what currently exists.

## Proposed topology

```mermaid
flowchart TB
    INET([Internet])
    ROUTER["Home Router\n(DHCP + NAT)"]
    HOST["Proxmox VE Host\nHP ProDesk 600 G1\n192.168.1.10 (static)"]

    INET --- ROUTER
    ROUTER --- HOST

    subgraph Proxmox["Proxmox Host"]
        HOST --- VM1["VM/LXC: Jellyfin\n192.168.1.11 (static)"]
        HOST --- VM2["VM: Docker host\n192.168.1.12 (static)"]
        HOST --- VM3["LXC: Pi-hole/AdGuard\n192.168.1.13 (static)\nnetwork DNS server"]
        HOST --- VM4["VM/LXC: WireGuard\n192.168.1.14 (static)\nVPN endpoint"]
    end

    subgraph DockerHost["Docker host (VM2) containers"]
        VM2 --- C1["Minecraft server"]
        VM2 --- C2["Grafana"]
        VM2 --- C3["Prometheus"]
    end

    CLIENT(["Remote client\n(phone/laptop)"]) -. WireGuard tunnel .-> VM4
```

## Assumptions baked into this proposal

- Flat network (no VLAN segmentation) — see the segmentation discussion in `networking.md` for
  why this is a reasonable starting default rather than a hard recommendation.
- Subnet assumed as `192.168.1.0/24` — replace with whatever the actual router's subnet is.
- Static IP reservations for anything acting as a server (Jellyfin, the Docker host, Pi-hole/
  AdGuard, the WireGuard endpoint) — set these via DHCP reservations on the router or static
  config on each VM/LXC, not left on dynamic DHCP, since Pi-hole in particular needs a fixed
  address to be usable as the network's DNS server.
- Docker workloads (Minecraft, Grafana, Prometheus) shown as containers inside a single "Docker
  host" VM, per the recommendation in `../docs/services.md` (Docker in a VM, not nested in LXC).

## To confirm/replace once known

- [ ] Actual router model and subnet
- [ ] Actual Proxmox host IP
- [ ] Whether reservations are done via router DHCP or static config per guest
- [ ] Any VLAN/segmentation decision, if it changes from the flat-network default above
