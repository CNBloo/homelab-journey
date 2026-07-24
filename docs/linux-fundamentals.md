# Linux Fundamentals

README lists "Learn Linux fundamentals" as in-progress, but nothing was written down for it yet.
This is a starting reference — the concrete commands/concepts worth being comfortable with before
(and while) working through Docker, Pi-hole, WireGuard, backups, and monitoring, since all of them
assume basic shell comfort.

## Filesystem hierarchy (what matters day-to-day)

| Path | What lives there |
|---|---|
| `/etc` | System-wide config files (most services you'll touch live here) |
| `/var/log` | Log files — first place to check when something's wrong |
| `/home/<user>` | Your user's files |
| `/opt` | Third-party/manually-installed software (some Docker Compose setups land config here) |
| `/dev` | Device files — e.g. `/dev/dri` matters for Jellyfin hardware transcoding (see `hardware.md`) |
| `/proc`, `/sys` | Virtual filesystems — live kernel/process/hardware info, not real files on disk |

## Permissions

Every file has an owner, a group, and permission bits for owner/group/others (read/write/execute).

- `ls -la` shows permissions as `rwxr-xr-x` style strings.
- `chmod 755 file` — numeric mode: read=4, write=2, execute=1, summed per owner/group/other
  (7 = rwx, 5 = r-x, etc).
- `chown user:group file` — change ownership.
- **This matters concretely for SSH key auth**: `~/.ssh` must be `700` and
  `~/.ssh/authorized_keys` must be `600`, or `sshd` silently refuses to use the key (see
  `troubleshooting.md`).

## systemd / systemctl

Almost everything long-running on a modern Linux server (Docker daemon, Jellyfin if installed as
a service, Pi-hole's FTL process, sshd itself) is managed by `systemd` as a "unit."

- `systemctl status <unit>` — is it running, and the last few log lines
- `systemctl start|stop|restart <unit>`
- `systemctl enable <unit>` — start automatically on boot
- `journalctl -u <unit>` — full logs for that unit
- `journalctl -u <unit> -b` — logs since the last boot only
- `journalctl -f` — follow logs live (like `tail -f` but for the whole journal)

This is the first thing to reach for whenever a service "isn't working" — status + logs, before
assuming a config problem.

## Package management

Proxmox VE is Debian-based, so its host and most Debian/Ubuntu-based VMs/containers use `apt`:

- `apt update` — refresh the package index (does not install/upgrade anything by itself)
- `apt install <package>`
- `apt upgrade` — upgrade installed packages
- `apt list --installed` — what's on the system

## SSH

- Key-based auth (recommended over passwords): generate a keypair with `ssh-keygen`, put the
  public key in the remote host's `~/.ssh/authorized_keys`.
- `ssh -v user@host` — verbose output, useful for debugging auth failures.
- `ssh-copy-id user@host` — copies your public key to a remote host's authorized_keys for you.

## Process management

- `ps aux` — every running process
- `top` / `htop` (if installed) — live view, useful for spotting a runaway process eating CPU/RAM
- `kill <pid>` / `kill -9 <pid>` — terminate a process (the `-9` is a hard kill, last resort since
  it skips the process's own cleanup)

## Disks and storage

- `lsblk` — list block devices (disks/partitions) in a tree view
- `df -h` — filesystem usage, human-readable
- `du -sh <dir>` — how much space a specific directory is using

## Cron (scheduled tasks)

Relevant later for backups (`vzdump` cron jobs) — `crontab -e` opens the current user's crontab;
format is `minute hour day month weekday command`, e.g. `0 3 * * *` = every day at 3am.

## Next steps for this doc

- [ ] Note the actual distro/version running on the Proxmox host and any VMs once checked
      (`cat /etc/os-release`)
- [ ] Record any command or concept that caused real confusion while working through Docker/Pi-hole/
      VPN setup — that's more valuable here than a generic command list
