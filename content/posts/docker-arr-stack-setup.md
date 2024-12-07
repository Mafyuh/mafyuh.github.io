+++
date = 2024-02-23T00:13:40Z
description = ""
draft = false
slug = "docker-arr-stack-guide"
title = "Docker Compose Arr Stack Guide"
tags = ["Homelab"]
comments = true
+++

This guide is for someone who is looking to setup an Arr Stack for media organization and downloading. This guide requires no remote path mappings, follows Trash-Guides recommendations and every command needed is copy-pasteable. The VM's in this guide are hosted on Proxmox 8.2.2, but you can use any Ubuntu environment (WSL-2, VirtualBox, etc.)

Arr VM Specs:
- 2 core host
- 8GB RAM
- 350GB Storage

### Prerequisites
- Ubuntu 22.04
- Any Usenet Server Subscription (preferred)
- Any Usenet Indexer Subscription (preferred)
- Real-Debrid Subscription (if you like torrents being fast)
- VPN Subscription (Bare minimum needed to download torrents)


## Folder Structure Setup
Run this command to make all folders, following [TRASH-guides recommended naming scheme](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/):
```
sudo mkdir -p /data/torrents/{books,movies,music,tv} /data/usenet/{incomplete,complete/{books,movies,music,tv}}
```

## Mounting NAS
I use my NAS for storing all my content, this allows me to have 1 spot to have everything saved too, and not getting tripped up with different file systems. You do not need a NAS, and can just skip this part of guide and use the local filesystem. I use TrueNAS Scale with NFS. In order to mount NFS shares to Docker Containers we need to install NFS on the host system:
```
sudo apt install nfs-common -y
```
### Install Docker

Now we have to install Docker, I use this command to install Docker and Docker Engine:
```
curl -fsSL https://get.docker.com | sudo sh
```
Now that docker is installed, we can add our user to the docker group so we don't have to use sudo every command:
```
sudo usermod -aG docker $USER
```
Now I would make a docker directory to store all your appdata, you can use your home directory if you want, but trash-guides recommend not doing so:
```
sudo mkdir -p /docker/appdata/{radarr,sonarr,bazarr,prowlarr,lidarr,sabnzbd,qbitty,rdt}
```
Then set permissions on the docker directory:
```
sudo chown -R $USER:$USER /docker
sudo chmod -R a=,a+rX,u+w,g+w /docker
```

I would restart the VM at this point so we don't have to use `sudo` for docker commands anymore.

## Docker Compose Files
Now that everything is setup, we can actually install the services:

This is a full docker compose file for pretty much all major Arr's and downloaders I use. Just remove whatever isn't needed in your setup. 

```
services:
# See https://docs.linuxserver.io/images/docker-sabnzbd/
  sabnzbd:
    image: lscr.io/linuxserver/sabnzbd:latest
    container_name: sabnzbd
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/sabnzbd:/config
      - /data/usenet:/data/usenet:rw
    ports:
      - 8080:8080
    restart: unless-stopped

# Downloading Torrents with VPN, see https://github.com/binhex/arch-qbittorrentvpn
  arch-qbittorrentvpn:
    image: binhex/arch-qbittorrentvpn:latest
    container_name: qbittorrentvpn
    volumes:
      - '/docker/appdata/qbitty:/config'
      - '/data/torrents/:/data/torrents'
      - '/etc/localtime:/etc/localtime:ro'
    ports:
      - '49550:49550'
      - '49551:8118'
    environment:
      - VPN_ENABLED=yes
      - VPN_PROV=protonvpn
      - VPN_CLIENT=wireguard
      - VPN_USER=username+pmp
      - VPN_PASS=
      - STRICT_PORT_FORWARD=yes
      - LAN_NETWORK=10.0.0.0/24
      - ENABLE_PRIVOXY=yes
      - PUID=1000
      - PGID=1000
      - WEBUI_PORT=49550
      - UMASK=1000
      - DEBUG=false
    cap_add:
      - NET_ADMIN
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    privileged: true
    network_mode: bridge
    restart: unless-stopped

# See https://github.com/rogerfar/rdt-client
  rdtclient:
    container_name: rdtclient
    volumes:
      - '/data/torrents:/data/torrents'
      - '/docker/appdata/rdt:/data/db'
    image: rogerfar/rdtclient
    restart: always
    logging:
      driver: json-file
      options:
        max-size: 10m
    ports:
      - '6500:6500'

# See https://docs.linuxserver.io/images/docker-bazarr/
  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    ports:
      - "6767:6767"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/bazarr:/config
      - nas:/data/media
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

#See https://docs.linuxserver.io/images/docker-lidarr/
  lidarr:
    image: lscr.io/linuxserver/lidarr:latest
    ports:
      - "8686:8686"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/lidarr:/config
      - /data:/data
      - nas:/data/media
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

# See https://docs.linuxserver.io/images/docker-prowlarr/
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    ports:
      - "9696:9696"
    volumes:
      - /docker/appdata/prowlarr:/config
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

# See https://docs.linuxserver.io/images/docker-radarr/
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    ports:
      - "7878:7878"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/radarr:/config
      - /data:/data
      - nas:/data/media
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

# See https://docs.linuxserver.io/images/docker-sonarr/
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    ports:
      - "8989:8989"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/sonarr:/config
      - /data:/data
      - nas:/data/media
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

networks:
  default:
    name: arrs

volumes:
  nas:
    driver: local
    driver_opts:
      type: nfs
      o: addr=<your-nas-ip>,vers=4,rw
      device: ":/path/to/your/nas/media"
```

Just change your nas IP address and the path to your media folder inside your nas. You may need to change your version depending on what version of nfs server you are running.

If you run into any apparmor errors, just add this to the end of the service that is using the nas Docker volume:
```
security_opt:
      - apparmor:unconfined
```
Example:
```
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    ports:
      - "8989:8989"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/sonarr:/config
      - /data:/data
      - nas:/data/media
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
    security_opt:
      - apparmor:unconfined
```

Note: Tagging to latest will work if you just plan on using `docker compose pull` to update your containers. But if you use tools like Renovate to keep these up to date you will want to tag to specific versions. If you want to see my actual homelab docker compose file its [here](https://git.mafyuh.dev/mafyuh/iac/src/branch/main/docker/arrs/docker-compose.yml)
## Running Docker Compose Files
In order to run these files, just copy the compose file and create a new docker-compose.yml file with:
```
nano docker-compose.yml
```
Paste in the content, CTRL + X to exit nano, Y to save, ENTER to keep filename. Then run:
```
docker compose up -d
```
## Conclusion
Congratulations on setting up your media library backend! We now have to go and configure all these services to work together, which I have another full blog post on which you can find [here](https://mafyuh.com/posts/arr-stack-config-guide).
