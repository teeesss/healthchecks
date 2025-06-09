# Healthchecks for Docker Compose

Docker healthchecks configured for specific containers in a Docker Compose setup. These were tested on a Linux system and should work across most Linux environments, including Synology NAS. Use `127.0.0.1` for healthchecks where possible, or the container's assigned Docker IP if needed. Some containers, like Jellyseerr, Unpackerr, and Watchtower, lack `curl`, requiring alternative healthcheck methods. Adjust configurations to match your specific ports and use case. These healthchecks enable the AutoHeal container to monitor and restart containers as needed. Watchtower is used for container updates but is outside this document's scope.

## Prerequisites

- Ensure all containers have the following label for AutoHeal compatibility:
```
  labels:
    - autoheal=true
```
#### Verify port mappings and static IPs match your setup.

#### Set necessary environment variables in a .env file.

```
Static IPs for Services

Docker GW:         172.50.0.1
VPN:               172.50.0.2
Emby:              172.50.0.3
Jellyseerr:        172.50.0.4
Sonarr:            172.50.0.5
Radarr:            172.50.0.6
Prowlarr:          172.50.0.7
Unpackerr:         172.50.0.8
Flaresolverr:      172.50.0.10
Choco:             172.50.0.50
Watchtower:        172.50.0.110
Nebula-sync:       172.50.0.125
PiHole:            (Not assigned in network)
```
## Container Healthchecks
```
vpn:
  image: thrnz/docker-wireguard-pia:latest
  healthcheck:
    test: ["CMD-SHELL", "/etc/healthcheck/vpn_healthcheck.sh"]
    interval: 60s
    timeout: 90s
    retries: 2
    start_period: 40s
```

### Emby
```
emby:
  image: emby/embyserver:latest
  healthcheck:
    test: ["CMD-SHELL", "netstat -lnt | grep -q ':8096'"]
    interval: 2m
    timeout: 10s
    retries: 5
    start_period: 90s
```
### Jellyseerr
```
jellyseerr:
  image: fallenbagel/jellyseerr:latest
  healthcheck:
    test: ["CMD-SHELL", "netstat -lnt | grep -q ':5055'"]
    interval: 2m
    timeout: 10s
    retries: 3
    start_period: 60s
```
### Sonarr
```
sonarr:
  image: lscr.io/linuxserver/sonarr:latest
  healthcheck:
    test: ["CMD", "curl", "-f", "http://127.0.0.1:8999/ping"]
    interval: 5m
    timeout: 20s
    retries: 3
    start_period: 40s
```
### Radarr
```
radarr:
  image: lscr.io/linuxserver/radarr:latest
  healthcheck:
    test: ["CMD", "curl", "-f", "http://127.0.0.1:7878/ping"]
    interval: 5m
    timeout: 20s
    retries: 3
    start_period: 42s
```
### Prowlarr
```
prowlarr:
  image: lscr.io/linuxserver/prowlarr:latest
  healthcheck:
    test: ["CMD", "curl", "-f", "http://127.0.0.1:9696/ping"]
    interval: 5m
    timeout: 20s
    retries: 3
    start_period: 44s
```
### qBittorrent
```
qbittorrent:
  image: lscr.io/linuxserver/qbittorrent:latest
  healthcheck:
    test: ["CMD", "curl", "-f", "http://127.0.0.1:${QBIT_WEBUI_PORT}/api/v2/app/version"]
    interval: 5m
    timeout: 20s
    retries: 3
    start_period: 45s
```
### SABnzbd
```
sabnzbd:
  image: lscr.io/linuxserver/sabnzbd:latest
  healthcheck:
    test: ["CMD", "curl", "-f", "http://127.0.0.1:6790/sabnzbd/api?mode=version&output=json"]
    interval: 5m
    timeout: 20s
    retries: 3
    start_period: 40s
```
### Flaresolverr
```
flaresolverr:
  image: ghcr.io/flaresolverr/flaresolverr:latest
  healthcheck:
    test: ["CMD-SHELL", "curl -s --fail http://localhost:8191 -o /dev/null"]
    interval: 2m
    timeout: 10s
    retries: 3
    start_period: 60s
```
### Unpackerr
```
unpackerr:
  image: golift/unpackerr:latest
  healthcheck:
    test: ["CMD", "/unpackerr", "--version"]
    interval: 10m
    timeout: 30s
    retries: 3
    start_period: 44s
```
### AutoHeal
```
autoheal:
  image: willfarrell/autoheal:latest
  healthcheck:
    test: ["CMD-SHELL", "pgrep -f autoheal || exit 1"]
    interval: 5m
    timeout: 10s
    retries: 3
    start_period: 35s
```
### Choco
```
choco:
  image: chocolatey/choco:latest-linux
  healthcheck:
    test: ["CMD-SHELL", "grep -q 'tail' /proc/[0-9]*/cmdline"]
    interval: 5m
    timeout: 10s
    retries: 3
    start_period: 10s
```
### Watchtower
```
watchtower:
  image: containrrr/watchtower:latest
  healthcheck:
    test: ["CMD", "/watchtower", "--health-check"]
    interval: 5m
    timeout: 30s
    retries: 3
    start_period: 35s
```
### PiHole
```
pihole:
  image: mpgirro/pihole-unbound:latest
  healthcheck:
    test: ["CMD-SHELL", "dig @127.0.0.1 -p 5335 google.com || exit 1"]
    interval: 1m
    timeout: 10s
    retries: 3
    start_period: 60s
```
### AutoHeal Configuration
```autoheal:
  image: willfarrell/autoheal:latest
  container_name: autoheal
  restart: always
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
  environment:
    - TZ=${TIMEZONE}
    - AUTOHEAL_CONTAINER_LABEL=all
    - AUTOHEAL_INTERVAL=60
    - AUTOHEAL_START_PERIOD=60
    - AUTOHEAL_DEFAULT_STOP_TIMEOUT=30
    - AUTOHEAL_DELAY=20
    - DOCKER_SOCK=/var/run/docker.sock
    - CURL_TIMEOUT=35
```
