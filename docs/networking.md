# Networking

## Current state

`TBD` — nothing here yet. Fill in once known:
- Router make/model and whether it supports VLANs
- Current subnet (e.g. `192.168.1.0/24`)
- DHCP range and whether the Proxmox host / VMs have static IPs or reservations
- Whether the homelab sits on the same network as regular household devices, or is segmented

See [`diagrams/network-diagram.md`](../diagrams/network-diagram.md) for a proposed topology to
compare against whatever the actual setup turns out to be.

## Segmentation

Worth deciding early, since Pi-hole (becomes the network's DNS server) and a VPN (needs a
routable target) both depend on this: keep the homelab on the same flat network as household
devices (simplest, fine for a single learner/household), or put it on its own VLAN (more setup,
but isolates homelab experiments from things like phones/laptops/smart TVs if something gets
misconfigured). For a single-host learning setup like this one, starting flat and segmenting
later if it becomes a real concern is a reasonable default — most home routers also don't support
VLANs without replacing them (e.g. with something running OPNsense/pfSense/a managed switch).

## VPN: WireGuard vs Tailscale

This is the open decision from the README roadmap ("Configure a VPN (WireGuard or Tailscale)").

### How each actually works

**WireGuard** is a protocol, not a service — it's just a fast, modern kernel-level tunnel.
Using it directly means you:
- Generate a public/private keypair for the server and for each client
- Manually define each peer's `AllowedIPs` (which traffic routes through the tunnel) and
  `Endpoint` (how peers find each other)
- Handle NAT traversal yourself — if your home router does CGNAT or you're behind a strict NAT,
  this can require port forwarding or a relay
- Own the whole config file (`/etc/wireguard/wg0.conf`) — no external account, no third party

**Tailscale** is a mesh VPN product built *on top of* WireGuard. It adds a coordination server
(Tailscale's own, hosted — or self-hostable via Headscale) that:
- Distributes keys and peer info automatically — no manual keypair/config management
- Handles NAT traversal transparently (DERP relays as fallback when direct connections fail)
- Gives every device a stable identity via your Tailscale account (SSO login), with per-device
  revocation from a dashboard

### Comparison

| | WireGuard (raw) | Tailscale |
|---|---|---|
| Setup effort | Manual config per peer | `tailscale up` on each device, done |
| What you learn | Tunneling, key exchange, routing, NAT traversal | Mesh VPN operations, less protocol depth |
| Self-hosted / no third party | Yes, fully | No (unless self-hosting Headscale) |
| NAT traversal | Manual/your problem | Automatic |
| Multi-device revocation | Manual (edit config, remove peer) | One click in dashboard |
| Cost | Free | Free tier is generous for personal use |

### Recommendation

Since this repo is explicitly a *learning* journal (not "get remote access with minimum effort"),
set up **raw WireGuard first** — this is where the actual networking learning happens (keypairs,
routing, `AllowedIPs`, opening the right UDP port on the router). Once that's working and
understood, try **Tailscale** on top/alongside to feel the ergonomics difference. Don't skip
straight to Tailscale — it would solve the immediate remote-access need but skip the protocol-level
understanding that's the stated point of this project.

## Next steps

- [ ] Fill in current-state section above (subnet, router, DHCP)
- [ ] Decide flat vs segmented network
- [ ] Stand up WireGuard manually, document the config and any port-forwarding needed
- [ ] Try Tailscale afterward, compare notes here
