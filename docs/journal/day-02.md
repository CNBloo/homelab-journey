# Day 2 - Storage Upgrade and Planning the Next Service
**Date:** July 23, 2026

## Objective
Upgrade the server's storage setup and start planning which self-hosted service to deploy next.

## Hardware Update
- Added a new SSD to the server
- Original HDD retained as secondary storage

## What I Accomplished
- Added a new SSD and migrated the OS over to it, setting it as the main boot drive
- Reformatted the original HDD so it's now available as usable storage on the Proxmox server
- Started reading through the Docker docs to decide what to deploy next — currently weighing an ad blocker (Pi-hole/AdGuard Home) against a VPN as the first containerized service

## Challenges
- Migrating the OS to a new boot drive without losing my existing Proxmox setup
- Reformatting and reintegrating the old HDD as usable storage

## What I Learned
- How to move a Proxmox boot drive over to a new SSD
- The basics of reformatting a drive and bringing it back online as storage in Proxmox
- Started building a mental model of Docker before installing it, so I go in with a plan instead of guessing

## Lessons Learned
Storage work turned out to be more foundational than I expected — getting the boot drive and secondary storage sorted properly now should make everything I build on top of it (Docker, Pi-hole/VPN, eventually Jellyfin) much smoother later.

## Next Steps
- Decide between Pi-hole/AdGuard Home and a VPN as the first Docker service to deploy
- Install Docker
- Hold off on rebuilding Jellyfin until I'm back at school, so I can pull free movies from the library for the media library
