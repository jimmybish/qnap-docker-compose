# Jimmy's QNAP Docker App Configuration
I wanted to make all my apps on my QNAP NAS portable so they're easy to restore or migrate to another device. Sonarr, Radarr and NZBGet were easy enough to build from scratch, but Plex needed a migration if I wanted to keep my library configuration and watched/unwatched statuses. Once the containers are up and running, follow the steps in [Migrate Plex QPKG to Docker.md](https://github.com/jimmybish/qnap-docker-compose/blob/main/Migrate%20Plex%20QPKG%20to%20Docker.md#migrate-plex-from-qpkg-to-docker) to complete the migration.

Otherwise, the below information is fine for a fresh installation as well.

## Containers Used
* https://hub.docker.com/r/linuxserver/sonarr
* https://hub.docker.com/r/linuxserver/radarr
* https://hub.docker.com/r/linuxserver/nzbget
* https://hub.docker.com/r/linuxserver/plex

## Required Apps
If creating these containers on a QNAP, add the [QNAPClub repo](https://www.qnapclub.eu/en) and ensure you can view the contents in App Center. Then install the following:
* **Qgit** - Git client to clone this repo.

## Steps and Container Doco
### Clone this Repository and Configure the Startup Script
You'll need to find a folder to store config in, as data stored in `~` won't survive a reboot (I learned that the hard way!). I chose the `/share/Container` fileshare created by Container Station to store both the `docker-compose.yml` config as well as each container's config folder.
```
cd /share/Container
git clone https://github.com/jimmybish/qnap-docker-compose.git
cd qnap-docker-compose
```

### Map the appropriate folders with those on the host:
Edit the config to suit your folder locations or create shared folders where defined in the config. The folder before `:` is the folder path on the NAS, after `:` is the folder inside the container.

**Folders required:**

```
# NZBGet
  - /share/Container/nzbget:/config   # App configuration data
  - /share/Download/NZBGet:/downloads # Parent downloads folder (will create subfolders for categories)

# Radarr
  - /share/Container/radarr:/config   # App configuration data
  - /share/Multimedia/Movies:/movies  # Folder on the NAS where movies are stored (Where Plex reads it from)
  - /share/Download/NZBGet/completed/Movies:/downloads  # Folder where completed movie downloads are stored (Where NZBGet drops them)

# Sonarr
  - /share/Container/sonarr:/config   # App configuration data
  - /share/Multimedia/TV:/tv          # Folder on the NAS where TV episodes are stored (Where Plex reads it from)
  - /share/Download/NZBGet/completed/TV:/downloads  # Folder where completed TV downloads are stored (Where NZBGet drops them)

# Plex
  - /share/Container/plex:/config      # App configuration data
  - /share/Multimedia/TV:/tv           # Folder on the NAS where TV episodes are stored
  - /share/Multimedia/Movies:/movies   # Folder on the NAS where movies are stored
  - /share/Multimedia/Pictures:/photos # Folder on the NAS where photos are stored
  - /share/Backups/plex_db:/db_backups # Folder on the NAS for automatic database backups
```

### User accounts and permissions
Containers are configured to run as the `docker-plex` user, which is also a member of the `docker` group (in case I want to create containers that run as other users). I created both via the QNAP Web UI, but confirmed the IDs in the CLI.
```
$ id docker-plex
uid=1001(docker-plex) gid=100(everyone) groups=100(everyone),1000(docker)

# From the yml:
      - PUID=1001
      - PGID=1000
```
All app folders are owned by `docker-plex:docker`, while the `docker` group has RW access to the media shares.

You will need to create a user and group and map to your user's IDs in each container config if you're like me and keep things separate and nailed down. Otherwise, set both the `PUID` and `PGID` values to 0, which is the `admin` user.

### Network
All containers except Plex use the standard NAT configuration, since they only require a single incoming port to the web interface. I've kept the port as the default for both inside and outside the container to keep it simple.
Plex uses quite a few more ports - both TCP and UDP. Claiming ownership of the media server in a NAT'ted container can also be a PITA since the incoming connection needs to be on the same subnet. You can Google something like `Plex docker claim server SSH tunnel` and check the [container documentation](https://hub.docker.com/r/linuxserver/plex) for required ports if you want to chase that path. I just set `network_mode: host` to make it all just work.

## Controlling the Containers
`Docker-compose` must be run from the folder containing `docker-compose.yml`. If not, the full path must be specified with the `-f` option.

### Starting and Applying Updated Config
```
docker-compose up -d
```
This will ensure all containers are in a running state. If any are already running, nothing will happen to them, but any others will be downloaded/reconfigured/started, as required. 

### Stopping
```
docker-compose stop plex
```
This will stop just the Plex container. Replace with the name of any other to stop that container.

### Updating
```
docker-compose pull
docker-compose up -d
```
This will update all containers to their latest version. Specify the name of a container to update only that one.

### Tail logs
```
docker logs -f plex
```
Tail logs for Plex. Replace with the name of any other to tail logs within that container.

### Run Bash in a Container (control from within)
```
docker exec -it radarr /bin/bash
```
The above enters you into a bash session inside the container for troubleshooting things like drive mounts, permissions, and so on. Use Ctrl+D or `exit` to drop back out of the container's shell.
