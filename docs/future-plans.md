# Future Plans / Roadmap Sequencing

The README lists the planned items as a flat checklist. This doc captures the *order* they make
sense in and *why*, since several depend on earlier ones being done first.

1. **Finish Jellyfin + Linux fundamentals** (already in progress) — see
   [`services.md`](./services.md) for the Jellyfin checklist. Linux fundamentals worth targeting:
   filesystem hierarchy, `systemctl`/systemd basics, package management, SSH key-based auth,
   `journalctl`. Every step below assumes comfort with a Linux shell.

2. **Document hardware + networking** — see [`hardware.md`](./hardware.md) and
   [`networking.md`](./networking.md). Needs to happen before Pi-hole/VPN since both depend on
   knowing the current subnet and having a static-IP plan.

3. **Docker** — see the Docker-on-Proxmox deep-dive in `services.md`. Recommendation: Docker in
   a VM, not nested in an LXC, for reliability over a small RAM saving.

4. **Pi-hole or AdGuard Home** — see comparison in `services.md`. Needs a static IP from step 2.

5. **VPN — WireGuard first, then Tailscale** — see the deep-dive in `networking.md`. WireGuard
   manually for the protocol-level learning; Tailscale afterward for the ergonomics comparison.

6. **Backups** — before adding stateful services worth losing (Minecraft world data, media
   library configs). Options to research: Proxmox Backup Server (PBS) for VM/CT-level snapshots,
   `vzdump` cron jobs as a simpler built-in alternative, or restic/borg for file-level backups of
   specific data. This matters more given the current single-HDD setup noted in `hardware.md` —
   no on-disk redundancy yet, so backups are the only protection against data loss until a NAS
   with RAID exists.

7. **Minecraft server** — straightforward once Docker exists; see `services.md` for the image
   to use.

8. **Grafana + Prometheus monitoring** — once 2-3 services are running so there's something to
   monitor. `node_exporter` + Prometheus + Grafana, using existing community dashboards for
   Proxmox/Pi-hole/Jellyfin rather than building from scratch.

9. **NAS build** — the biggest step. Decide first: separate physical box (TrueNAS/Unraid) vs. a
   VM/LXC on the existing Proxmox host with passed-through disks. Either way, research ZFS basics
   before buying drives — the redundancy level chosen determines the drive count needed:
   - **Mirror** (like RAID 1): simplest, 50% capacity efficiency, tolerates 1 drive failure
   - **RAIDZ1** (like RAID 5): better capacity efficiency, tolerates 1 drive failure, needs 3+ drives
   - **Stripe** (like RAID 0): no redundancy at all, don't use this for anything you'd miss

## Documentation habit

Keep filling in these topic docs as each area is actually worked on, rather than only writing
day-N journal entries — the journal is good for narrative/reflection, these docs are what's
actually useful to reference back to later (e.g. "what static IP did I give Pi-hole again?").
