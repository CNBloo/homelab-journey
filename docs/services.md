# Services

## Jellyfin

**Status:** installed, plugins explored, libraries not yet fully configured (in progress per
README roadmap).

Open items to close this out:
- [ ] Define library structure (e.g. separate libraries for Movies / TV / Music, correct
      folder layout so Jellyfin's metadata agents match content correctly)
- [ ] Pick and configure metadata agents/providers for each library
- [ ] Check hardware transcoding — see [`hardware.md`](./hardware.md) for the Quick Sync /
      `/dev/dri` passthrough note; without this, Jellyfin falls back to CPU transcoding, which
      the i5-4570 (4 cores, no Hyper-Threading) will struggle with with more than one
      simultaneous transcode
- [ ] Decide on remote access approach for Jellyfin once the VPN (see `networking.md`) is up,
      rather than exposing it directly to the internet

## Docker on Proxmox

Not yet installed (planned, next on the roadmap after Linux fundamentals/Jellyfin).

### The real decision: where does Docker run?

Proxmox gives three realistic options, and they're not equivalent:

1. **Docker inside a VM** — full hardware virtualization, Docker runs exactly like it would on
   bare metal. Fully isolated from the Proxmox host and other VMs. Costs a bit more RAM/overhead
   since it's a full guest OS, but there's no nesting weirdness and upgrades (Docker version,
   kernel) are just normal Linux upgrades inside that VM.

2. **Docker inside an LXC container** — much lighter (shares the host kernel, near-zero
   overhead), but running Docker *inside* an LXC means nesting one container runtime inside
   another. This needs:
   - `features: nesting=1` set on the LXC config
   - Often `keyctl=1` as well
   - Privileged (not unprivileged) containers in some Proxmox/kernel version combos, which
     reduces isolation from the host
   - Known fragility around Proxmox/kernel upgrades breaking the nesting setup

3. **Skip Docker, use Proxmox's native LXC containers directly** — for single-service containers
   (e.g. just Pi-hole, just a game server) you don't strictly need Docker at all; Proxmox LXCs
   can run the service directly. You lose `docker-compose`'s multi-container orchestration and
   the huge library of ready-made Docker images, which is a real cost for stacks like
   Grafana+Prometheus that assume Docker.

### Recommendation

Use **Docker inside a VM**. Given this is a learning-focused single-host setup (not resource-
constrained), the reliability and simplicity of a fully isolated VM outweighs the RAM savings of
LXC nesting — and it avoids debugging nesting/cgroup issues that don't teach anything about
Docker itself, just about Proxmox's LXC internals. Revisit LXC-nested Docker later, specifically
as a *learning exercise* in its own right, once the basics are solid.

### Docker Compose

Since Pi-hole, Grafana, and Prometheus are all typically distributed as Docker images meant to
be run via `docker-compose.yml` (not one-off `docker run` commands), plan to write compose files
for each service and keep them in [`scripts/`](../scripts/) — this also gives the repo actual
version-controlled configuration instead of manual one-time setup steps.

## Pi-hole vs AdGuard Home

Open decision from the README roadmap.

| | Pi-hole | AdGuard Home |
|---|---|---|
| Blocklist ecosystem | Larger, more community lists (e.g. many curated lists on Firebog) | Smaller but overlapping, supports same list formats |
| UI | Functional, older-style | More modern, arguably cleaner |
| DNS-over-HTTPS/TLS upstream | Needs `cloudflared` or similar add-on | Built in |
| Ad blocking approach | DNS sinkholing | DNS sinkholing (same core mechanism) |
| Maturity | Older, very widely deployed | Newer, actively developed |

**Recommendation:** either is a reasonable choice — pick AdGuard Home if built-in DoH upstream
and a more modern UI matter; pick Pi-hole if leaning on the larger community blocklist ecosystem
matters more. Whichever is chosen needs a **static IP** (see `networking.md`) since it becomes
the network's DNS server — this is why networking should be documented before this step.

## Minecraft server

Not started. Once Docker is running, `itzg/docker-minecraft-server` is the standard community
image — handles Forge/Fabric/Paper/vanilla, EULA acceptance via env var, and RCON out of the box.
Revisit this doc once Docker exists to write the actual compose config used.

## Monitoring (Grafana + Prometheus)

Not started — intentionally sequenced after Docker/Pi-hole/Jellyfin exist, so there's something
real to monitor. When it's time: `node_exporter` for host-level metrics (CPU/RAM/disk on the
Proxmox host itself), Prometheus to scrape it, Grafana to visualize — and importing existing
community dashboards (Proxmox, Pi-hole, and Jellyfin all have published dashboard IDs) instead of
building panels from scratch.
