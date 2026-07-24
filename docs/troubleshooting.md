# Troubleshooting / Common Issues Reference

A running list of known, real-world gotchas for each stage of the roadmap — populated ahead of
time so there's somewhere to look when something breaks, rather than starting from zero mid-issue.
Add to this as things are actually hit (move confirmed fixes from "known pitfalls" into a
"confirmed on this box" note once verified).

## Proxmox

- **"No network card detected" / can't reach the web UI after install:** usually the onboard
  NIC needs a driver Proxmox's installer doesn't ship by default (common on older consumer
  boards like this ProDesk's). Check `ip a` from the Proxmox console directly — if the interface
  isn't listed at all, it's a driver issue, not a config issue.
- **VM won't boot after creation — "no bootable device":** almost always the boot order in the
  VM's Options tab doesn't have the right disk/CD-ROM first, or the ISO is still mounted as the
  primary boot device after install finished. Check Options → Boot Order.
- **Storage shows full unexpectedly:** Proxmox's default `local-lvm` thin-pool can look full in
  the UI while `df -h` on the host shows space free — this is thin-provisioning overcommit
  showing up. `pvesm status` gives the real picture per storage pool.

## Jellyfin

- **Hardware transcoding not being used despite Quick Sync being available:** the most common
  cause is `/dev/dri` not actually being passed into the container/VM Jellyfin runs in, or the
  `render` group permissions not matching between host and container. Verify inside the
  container: `ls -la /dev/dri` should show `renderD128`; then check Jellyfin's own transcoding
  logs (Dashboard → Logs) for `VAAPI` init failures, not just "check the settings checkbox."
- **Libraries scan but show wrong metadata:** almost always a folder-naming mismatch — Jellyfin's
  metadata agents expect `Movie Name (Year)/Movie Name (Year).ext` or
  `Show Name/Season 01/Show Name S01E01.ext` conventions. Loose/inconsistent naming is the #1
  cause of wrong-poster/wrong-match issues, not a Jellyfin bug.

## Linux fundamentals (general)

- **`systemctl` command "hangs"**: usually not actually hung — it's often waiting on a dependency
  timeout (commonly network-online.target on a slow DHCP negotiation at boot). `systemctl status
  <unit>` and `journalctl -u <unit> -b` are the first two commands to reach for before assuming
  something is broken.
- **SSH key auth "not working"**: the overwhelmingly common cause is permissions —
  `~/.ssh` must be `700` and `~/.ssh/authorized_keys` must be `600`; sshd silently refuses to use
  key auth if permissions are too open, with no obvious error unless you check
  `journalctl -u sshd` or run `sshd -T` / connect with `ssh -v` for verbose client-side output.

## Docker on Proxmox

- **Docker-in-LXC: containers fail to start with cgroup/AppArmor errors:** this is the
  nesting problem flagged in `services.md`'s Docker-on-Proxmox deep-dive. If choosing LXC despite
  the VM recommendation, the two settings almost always needed are `features: nesting=1` in the
  LXC config and switching from unprivileged to privileged (with the security tradeoff that
  implies) — and this can silently break again after a Proxmox host kernel upgrade.
  Running Docker in a VM (the recommended path here) avoids this class of issue entirely.
- **Docker networking conflicts with the LAN subnet:** Docker's default bridge network
  (`172.17.0.0/16`) can collide with a VPN's assigned subnet or a corporate/campus network if
  ever connecting from one — worth knowing `docker network create` lets you pick a non-conflicting
  subnet if this comes up.

## Pi-hole / AdGuard Home

- **DNS loop / "everything stops resolving" right after setup:** happens when the Pi-hole/AdGuard
  box's own upstream DNS is pointed back at itself (directly or indirectly through the router).
  Upstream DNS (e.g. `1.1.1.1`, `9.9.9.9`, or DoH) must be a *different* resolver than the ad-blocker
  itself.
- **Devices on the network still show ISP DNS / ads not blocked:** the ad-blocker's IP has to
  actually be pushed out as the DNS server — either via the router's DHCP settings (server-wide,
  simplest) or per-device — and this depends on having the static IP set up first per
  `networking.md`. A common trap: setting it in the router UI but the router's own DHCP lease
  time means existing devices don't pick up the change until they renew (or are rebooted).

## VPN (WireGuard / Tailscale)

- **WireGuard peer "handshake" never completes:** almost always one of: the router isn't actually
  forwarding the WireGuard UDP port to the host (double-NAT from an ISP modem+router combo is a
  common silent cause), the client's `AllowedIPs` don't include the route needed to reach the
  server's actual LAN, or system clocks are wildly out of sync (WireGuard handshakes are time-
  sensitive).
- **Tailscale connects but traffic doesn't route to LAN devices, only the host itself:**
  Tailscale needs the host explicitly advertised as a **subnet router** (`tailscale up
  --advertise-routes=<subnet>`) and that route approved in the Tailscale admin console — this is
  a separate step from just installing Tailscale.

## Minecraft server (Docker)

- **Server appears to start but no one can connect:** check the port mapping in the compose file
  actually maps the container's `25565` to a host port, and that the router forwards that same
  port if connecting from outside the LAN — this is the most common cause, more often than an
  actual server misconfiguration.
- **World/player data lost after a container restart:** means the world directory wasn't mounted
  as a persistent volume — this needs to be caught *before* players build anything, since it's
  unrecoverable after the fact without a backup.

## Backups

- **`vzdump` backup completes but restore fails:** check available storage space on the backup
  target *before* trusting a backup exists — `vzdump` can report success while silently truncating
  if the target ran out of space mid-backup, depending on version/config.
- **Restic/borg "repository locked" errors:** usually a previous run was killed uncleanly (e.g.
  host rebooted mid-backup); both tools have an explicit `unlock`/break-lock command rather than
  needing to delete repository state.

## Grafana + Prometheus

- **Prometheus target shows "down" for `node_exporter`:** almost always either the exporter isn't
  actually running (`systemctl status node_exporter`) or a firewall is blocking its default port
  `9100` between the Prometheus host and the target.
- **Grafana dashboard shows "No data" despite Prometheus having data:** the most common cause is
  the datasource UID in an imported community dashboard JSON not matching the datasource actually
  configured in this Grafana instance — re-select the datasource per panel, or fix it at import
  time via the datasource-mapping step Grafana's import UI offers.

## ZFS / NAS

- **Pool created with wrong `ashift` value:** `ashift` (sector size alignment) is set at pool
  creation and **cannot be changed without destroying and recreating the pool** — for modern
  drives (4K native sector size), `ashift=12` is almost always correct; getting this wrong
  silently costs real performance rather than causing an obvious error, which is why it's worth
  getting right before creating the pool rather than after.
- **"Available space" lower than expected on a mirror/RAIDZ:** this is expected — ZFS reports
  usable space after redundancy overhead, not raw drive capacity; a two-drive mirror of 2x 4TB
  drives shows ~4TB usable, not 8TB, by design.
