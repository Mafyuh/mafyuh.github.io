+++
date = 2024-02-23T00:13:40Z
description = ""
draft = false
slug = "docker-arr-stack-guide"
title = "Docker Compose Arr Stack Guide"
tags = ["Homelab"]
comments = true
+++

This guide is for someone who is looking to setup an Arr Stack for media organization and downloading. This guide requires no remote path mappings, follows Trash-Guides recommendations and every command needed is copy-pasteable. The VM's in this guide are hosted on Proxmox 8.1.4, but you can use any Ubuntu environment (WSL-2, VirtualBox, etc.)

Arr VM Specs:
- 2 core host
- 8GB RAM
- 100GB Storage

Downloader VM Specs:
- 2 core host
- 4GB RAM
- 250GB Storage (can download up to this limit at a time, be careful when mass downloading or give plenty of space) 

### Prerequisites
- Ubuntu 22.04
- Any Usenet Server Subscription (preferred)
- Any Usenet Indexer Subscription (preferred)
- Real-Debrid Subscription (if you like torrents being fast)
- VPN Subscription (Bare minimum needed to download torrents)


## Folder Structure Setup
Run this command to make all folders, following [TRASH-guides recommended naming scheme](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/):
```
sudo mkdir -p /data/torrents/{books,movies,music,tv} /data/usenet/{incomplete,complete/{books,movies,music,tv}} /data/media/{books,movies,music,tv}
```

## Mounting NAS
I use my NAS for storing all my content, this allows me to have 1 spot to have everything saved too, and not getting tripped up with different file systems. You do not need a NAS, and can just skip this part of guide and use the local filesystem. I use TrueNAS Scale with SMB. In order to mount SMB shares to Linux filesystem we need to install CIFS:
```
sudo apt install cifs-utils -y
```
then we need to tell the system which directory to map it to, to do this: 
```
sudo nano /etc/fstab
```
at the end of the file, add an entry for your NAS as such:
```
//<NAS IP>/<NAS Share> /data/media cifs username=<user>,password=<pass>,uid=1000,gid=1000,auto,nofail 0 0
```
be sure to replace your credentials.

To mount your NAS, you can run:
```
sudo mount -a
```
then run the following to make sure your NAS is mounted: 
```
ls /data/media
```

Everything in your NAS should be showing now, but we need to set permissions, to do that run:
```
sudo chown -R $USER:$USER /data
sudo chmod -R a=,a+rX,u+w,g+w /data
```
### Install Docker

Now we have to install Docker, I use this command to install Docker and Docker Engine:
```
curl -fsSL https://get.docker.com | sudo sh
```
Now that docker is installed, we can add our user to the docker group so we dont have to use sudo every command:
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

## 2 VM Setup
I have my downloaders (Sab, Qbitty, Rdt-client) on a different VM than my ARR's, this is cause when I had everything on 1 docker host, I would have constant HTTP errors from Sab mainly, and as Sab is where I get most of my media, I decided to move to another VM, and then SMB share the download directories over to my ARR's VM. 

You do not have to do this, you can just have 1 docker host, up to you. It is alot less work to do all in one 1 VM.

If you do this, you need to replicate the origin setup, making all the same directories, then run:
```
sudo apt update
sudo apt install samba
```
We need to configure Samba to tell it what we are sharing:
```
sudo nano /etc/samba/smb.conf
```
Add the following at the end of this file:
```
[usenet]
  path = /data/usenet
  read only = no
  guest ok = no
  create mask = 0755

[torrents]
  path = /data/torrents
  read only = no
  guest ok = no
  create mask = 0755
```

To create your username and password, replace your_username with your actual username:
```
sudo smbpasswd -a your_username
```
Then restart samba with:
```
sudo systemctl restart smbd
```
Go back to your Arr VM and add the following to your /etc/fstab:
```
//<nas-ip>/usenet /data/usenet cifs username=<username>,password=<password>,uid=1000,gid=1000,auto,nofail 0 0
//<nas-ip>/torrents /data/torrents cifs username=<username>,password=<password>,uid=1000,gid=1000,auto,nofail 0 0
```
Mount them with:
```
sudo mount -a
```
Then re-run our permissions command:
```
sudo chown -R $USER:$USER /data
sudo chmod -R a=,a+rX,u+w,g+w /data
```

I would reboot this VM at this point, this will make sure it auto connects to our SMB shares at boot.
## Docker Compose Files
Now that everything is setup, we can actually install the services:

### One VM 
This is a full docker compose file for pretty much all major Arr's and downloaders I use. I threw Lidarr in here as well, as I run Lidarr for music, but if you dont care about music you can remove lidarr:
```
version: "3.9"

services:
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

  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    ports:
      - "6767:6767"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/bazarr:/config
      - /data/media:/data/media
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

  lidarr:
    image: lscr.io/linuxserver/lidarr:latest
    ports:
      - "8686:8686"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/lidarr:/config
      - /data:/data
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

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

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    ports:
      - "7878:7878"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/radarr:/config
      - /data:/data
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    ports:
      - "8989:8989"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/sonarr:/config
      - /data:/data
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

networks:
  default:
    name: arrs_default
```

### 2 VM
Arrs:
```
version: "3.7"
services:
  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    ports:
      - "6767:6767"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/bazarr:/config
      - /data/media:/data/media
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

  lidarr:
    image: lscr.io/linuxserver/lidarr:latest
    ports:
      - "8686:8686"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/lidarr:/config
      - /data:/data
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

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

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    ports:
      - "7878:7878"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/radarr:/config
      - /data:/data
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    ports:
      - "8989:8989"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/appdata/sonarr:/config
      - /data:/data
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000

networks:
  default:
    name: arrs_default
```

Downloaders:
As stated previously, Sab downloads most of my content (95%), you do not need all 3 of these, you can just copy the Sab part and just use Usenet with Sab. But I like to have a variety.

```
version: '3.9'
services:
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
```
## Running Docker Compose Files
In order to run these files, it depends on which option you chose, if 1 VM setup, just copy the compose file and create a new docker-compose.yml file with:
```
nano docker-compose.yml
```
Paste in the content, CTRL + X to exit nano, Y to save, ENTER to keep filename. Then run:
```
docker compose up -d
```
If you are using 2 VM's, you need to do this 2x. One for each docker-compose file.

## Conclusion
Congratulations on setting up your media library backend! We now have to go and configure all these services to work together, which I have another full blog post on which you can find here.
