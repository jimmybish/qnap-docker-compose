# qnap-docker-compose
Docker Compose config for the home QNAP NAS

## Clone the repository
```
git clone https://github.com/jimmybish/qnap-docker-compose.git
```

## Map the appropriate folders with those on the host:
Edit the config to suit your folder locations or create shared folders where defined in the config.

Folders required:

```
# NZBGet
  - /share/Container/nzbget:/config   # App configuration data
  - /share/Download/NZBGet:/downloads # Parent downloads folder (will create subfolders for categories)

# Radarr
  - /share/Container/radarr:/config   # App configuration data
  - /share/Multimedia/Movies:/movies  # Folder where movies are stored (Where Plex reads it from)
  - /share/Download/NZBGet/completed/Movies:/downloads  # Folder where completed movie downloads are stored (Where NZBGet drops them)

# Sonarr
  - /share/Container/sonarr:/config   # App configuration data
  - /share/Multimedia/TV:/tv          # Folder where TV episodes are stored (Where Plex reads it from)
  - /share/Download/NZBGet/completed/TV:/downloads  # Folder where completed TV downloads are stored (Where NZBGet drops them)

# Plex
  - /share/Container/plex:/config      #App configuration data
  - /share/Multimedia/TV:/tv           # Folder where TV episodes are stored
  - /share/Multimedia/Movies:/movies   # Folder where movies are stored
```

## User accounts and permissions
Containers are configured to run as the `docker-pirate` user, which is also a member of the `docker` group (in case I want to create containers that run as other users).
```
# id docker-pirate
uid=1004(docker-pirate) gid=100(everyone) groups=100(everyone),1000(docker)
```
All shared folders are owned by `docker-pirate:docker`.

## Controlling the containers
`Docker-compose` must be run from the folder containing `docker-compose.yml`. If not, the full path must be specified.

### Starting
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
```
This will update all containers to their latest version. Specify the name of a container to update only that one.

### Tail logs
```
docker-compose logs -f plex
```
Tail logs for Plex. Replace with the name of any other to tail logs within that container.
