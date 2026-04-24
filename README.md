# Homelab

This is my self-hosted media stack running on a ThinkCentre M710 series machine via Docker Compose, accessible over a NetBird VPN. Designed for clean service isolation by concern and utility.

## Architecture

All services are bound to the host's local interface. Remote access is handled by NetBird, which allows the services to be accessible on a private WireGuard mesh network and avoid exposing it to the public.

### Hardware
- ThinkCentre M710 Series
- External Hard Drive Bay - primary media storage

### Software
- [Docker](https://www.docker.com/) - Containerization, runs the services
- [NetBird](https://www.netbird.io/) - WireGuard based VPN for private access to services
- [Debian](https://www.debian.org/) - Operating system for the server.

## Services
- Media/
    - JellyFin/
        - [Jellyfin (port 8096)](https://hub.docker.com/r/jellyfin/jellyfin) - Media server, browser, and player
        - [slskd (port 5030)](https://hub.docker.com/r/slskd/slskd) - Client for SoulSeek, a p2p file sharing network
    - Starrs/ 
        - [qbittorrent (port 8080)](https://hotio.dev/containers/qbittorrent/) - BitTorrent client with VPN built in (I use Private Internet Access, but any VPN should work. Check the documentation for more details)
        - [Prowlarr (port 9696)](https://hotio.dev/containers/prowlarr/) - Indexer for qbittorrent and the other starr services below
        - [Lidarr (port 8686)](https://hotio.dev/containers/lidarr/) - Music collection manager
        - [Radarr (port 7878)](https://hotio.dev/containers/radarr/) - Movie collection manager
        - [Sonarr (port 8989)](https://hotio.dev/containers/sonarr/) - TV collection manager
        - [Seerr (port 5055)](https://hotio.dev/containers/overseerr/) - Better web interface for managing movies and tv shows, allows multiple users and request handling.
        - [FlareSolverr (port 8191)](https://github.com/FlareSolverr/FlareSolverr) - Cloudflare bypass for Prowlarr
- Apps/
    - Mealie
        - [Mealie (port 9000)](https://docs.mealie.io/) - Recipe manager and meal planner
- Utilities/
    - [filebrowser (port 8888)](https://hub.docker.com/r/hurlenko/filebrowser) - Web UI for managing files on the server

## Design Decisions

I separate services into folders grouped by purpose, typically to support or based on one main service. This keeps each grouping separated by concerns and allows me to easily restart, rebuild, or remove specific groupings of services or individual services themselves without affecting other running containers. 

Currently, I have qbittorrent in the starrs directory because the indexer (prowlerr) relies on it. I am contemplating moving it and slskd to their own directory and linking them via network instead, but am holding off for now since only the Starrs stack makes use of it.

Utilities is used strictly for services that help to manage the server and the stack itself and don't depend on / aren't interacted with any other service directly.

Since the project scale is relatively small and only runs on one node, I opted to use docker compose instead of kubernetes. If I begin to expand my hardware to add redundancy or want to assure more reliability, I might transition to kubernetes instead.

For the Starr stack, I use hotio.dev images because it was convenient for me when I was setting it up and because some services did not have an official docker image. The hotio.dev website provides a really simple list of common services you might see running together for this stack as well. You do not have to use them if you prefer others, especially since there isn't a reason to directly run a VPN on each of the services themselves (read [this article on running the apps with a VPN](https://wiki.servarr.com/en/vpn) for a more in-depth explanation), but they have worked well for me and I have no intention to change them.

I use NetBird as the VPN of choice because of its open source nature, ease of use, and its ability to be self hosted if I need to. NetBird allows all connected devices to access these services using the respective hosts (though I plan to add DNS in the future to reduce the amount of ports I have to remember).

## To Do
- [ ] Move `slskd` from `jellyfin` to it's own `download-clients` directory
- [ ] Create a simple makefile to easily start and stop all services
- [ ] Add DNS for the services to make them easier to find and connect to
