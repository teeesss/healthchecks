# healthchecks
Docker healthchecks that work for specific containers inside a docker compose file. These were all configured on a Synology NAS.
Have confirmed thes below healthchecks work at the time of this document. Use 127.0.0.1 if possible, otherwise selected docker IP for the container.
Some containers were missing curl, such as Jellyseerr, Unpackerr and Watchtower so had to resort to using other methods for a healthcheck to work successfully.
Update the configuration as for each container for your specific needs or use cases. AutoHeal configuration is at the bottom for reference.
Some of my ports might be different than the defaults. Ensure yours match your configuration.

```
###########################################################
####### Check and set any needed variables in .env  #######
###########################################################
```

### Containers should be configured with the label below

```
labels:
  - autoheal=true
```

Static IPs for each service listed here:
```
Docker GW         172.50.0.1
VPN               172.50.0.2
Embyserver        172.50.0.3
Jellyseerr        172.50.0.4
Sonarr            172.50.0.5
Radarr            172.50.0.6
Prowlarr          172.50.0.7
Unpackerr         172.50.0.8
Flaresolverr      172.50.0.10
Portainer         172.50.0.30
Syncthing         172.50.0.40
Openspeedtest     172.50.0.50
Speedtest-tracker 172.50.0.60
Unifi             172.50.0.70
Watchtower        172.50.0.110
```

```
  #########################
  ####### PIA VPN #########
  #########################

  vpn:
    image: thrnz/docker-wireguard-pia:latest
    container_name: vpn
    healthcheck:
      test: ["CMD", "ping", "-c", "1", "8.8.8.8"]
      interval: 1m
      timeout: 10s
      retries: 3
      start_period: 30s
```   

```
  ###########################
  ####### Embyserver ########
  ###########################

  embyserver:
    image: emby/embyserver:latest
    healthcheck:
      test: ["CMD-SHELL", "wget -q -O - http://localhost:8096/System/Ping | grep -q 'Emby Server' || exit 1"]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 45s
``` 

```
  ###########################
  ####### Jellyseerr ########
  ###########################

  jellyseerr:
    image: fallenbagel/jellyseerr:latest
    container_name: jellyseerr
    healthcheck:
      test: ["CMD", "node", "-e", "const http = require('http'); const options = { hostname: '127.0.0.1', port: 5055, path: '/api/v1/status', method: 'GET' }; const req = http.request(options, (res) => { if (res.statusCode == 200) { process.exit(0); } else { process.exit(1); }}); req.on('error', (e) => { process.exit(1); }); req.end();"]
      interval: 5m
      timeout: 30s
      retries: 3
      start_period: 45s
    depends_on:
      vpn:
        condition: service_healthy
```

```
  #########################
  ######## Sonarr #########
  #########################

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://127.0.0.1:8999/ping" ]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 2m
```

```
  #########################
  ######## Radarr #########
  #########################

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=${USER_ID}
      - PGID=${GROUP_ID}
      - TZ=${TIMEZONE}
    volumes:
      - ./radarr:/config
      - /${DOWNLOAD_ARR}:/data
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://127.0.0.1:7878/ping" ]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 30s
```

```
  #########################
  ####### Prowlarr ########
  #########################

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://127.0.0.1:9696/ping" ]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 45s
```

```
  ###########################
  ####### qBittorrent #######
  ###########################

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://127.0.0.1:${QBIT_WEBUI_PORT}/api/v2/app/version" ]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 35s
    depends_on:
      vpn:
        condition: service_healthy
```

```
  #########################
  ######## SABNZB #########
  #########################

  sabnzbd:
    image: lscr.io/linuxserver/sabnzbd:latest
    container_name: sabnzb
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://127.0.0.1:6790/sabnzbd/api?mode=version&output=json" ]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 45s
    depends_on:
      vpn:
        condition: service_healthy
```

```
  ###########################
  ####### Flaresolverr ######
  ###########################

  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://172.50.0.10:8191" ]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 50s
```

```
  #########################
  ####### Portainer #######
  #########################

    healthcheck:
      test: wget --no-verbose --tries=3 --spider http://127.0.0.1:9000 || exit 1
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 50s
```

```
  #########################
  ####### Unpackerr #######
  #########################

  unpackerr:
    image: golift/unpackerr:latest
    container_name: unpackerr
    healthcheck:
      test: [ "CMD", "/unpackerr", "--version" ]
      interval: 10m
      timeout: 20s
      retries: 3
      start_period: 35s
```

```
  ###########################
  ####### Watchtower ########
  ###########################

    healthcheck:
      test: ["CMD", "/watchtower", "--health-check"]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 30s
```

```
  ###########################
  ####### Syncthing #########
  ###########################

    healthcheck:
      test: ["CMD", "curl", "-f", "http://127.0.0.1:8384/rest/noauth/health"]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 50s
```

```
  ###############################
  ####### OpenSpeedTest #########
  ###############################

    healthcheck:
      test: ["CMD-SHELL", "curl -If http://127.0.0.1:3000 | grep 'HTTP/1.1 200 OK'"]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 40s
```

```
  ###################################
  ####### Speedtest Tracker #########
  ###################################

    healthcheck:
      test: ["CMD-SHELL", "curl -f http://127.0.0.1:80 || exit 1"]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 50s
```

```
  #######################
  ####### Unifi #########
  #######################

    healthcheck:
      test: ["CMD", "curl", "-fk", "https://127.0.0.1:8443/manage/account/login" ]
      interval: 5m
      timeout: 20s
      retries: 3
      start_period: 50s
```

```
  #########################
  ####### AutoHeal ########
  #########################

  autoheal:
    image: willfarrell/autoheal:latest
    container_name: autoheal
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - TZ=${TIMEZONE}
      - AUTOHEAL_CONTAINER_LABEL=all
      - AUTOHEAL_INTERVAL=30
      - AUTOHEAL_START_PERIOD=30
      - AUTOHEAL_DEFAULT_STOP_TIMEOUT=30
      - AUTOHEAL_DELAY=15
      - DOCKER_SOCK=/var/run/docker.sock
      - CURL_TIMEOUT=35
```
