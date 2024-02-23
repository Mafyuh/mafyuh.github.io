+++
date = 2024-02-22T00:13:40Z
description = ""
draft = false
slug = "selfhosted-game-servers"
title = "Selfhosted Game Servers"
tags = ["Homelab"]
comments = false
+++

Something I only got into recently is hosting video game servers for games that support servers. Maybe it's just something about having another server, cause these are totally not needed. But they are pretty easy to setup thanks to the open-source community.

# Sons of the Forest
I wanted to play sons one day and when I looked into multiplayer I seen there were options for servers. This sparked me Googling and finding this repo.

Setting this up took a bit, as the README was not very great. But I got it all figured out after reading GH Issues for who knows how long. Good old Linux permissions.

Here is a link to the repo I used
https://github.com/jammsen/docker-sons-of-the-forest-dedicated-server

VM Details
- Proxmox VM Ubuntu 22.04 Cloud image
- 4 core host
- 16GB RAM
- 100GB Storage

First I created a sons folder in my home directory and cd into it. To make the games directories I run:

```bash
mkdir game steamcmd winedata
```

My docker-compose is the same as on GH, but it is as follows:

```bash
version: '3.9'
services:
  sons-of-the-forest-dedicated-server:
    container_name: sons-of-the-forest-dedicated-server
    image: jammsen/sons-of-the-forest-dedicated-server:latest
    restart: always
    environment:
      ALWAYS_UPDATE_ON_START: 1
    ports:
      - 8766:8766/udp
      - 27016:27016/udp
      - 9700:9700/udp
    volumes:
      - ./steamcmd:/steamcmd
      - ./game:/sonsoftheforest
      - ./winedata:/winedata
```

This is in the sons folder.

Whenever I go and play I enable the port forward rules in my pfSense. Then once I or a friend get off I disable the forwards. The logs from the container do state when in sleep mode, so I am thinking of an automation that when in sleep mode it'll update my pfSense port forward. Maybe one day, but for now manually enable/disable. I do this as I dont want any port forwards on my network, if its just temporary like these it's fine, but never leave a port forward open to game services if its inside your home network.

# Palworld

When Palworld first came out I really wanted to mod actual Pokemon into the game, as I feel most of the Pals in the game look like AI generated garbage. But I'm no video game mod-dev and I dont see anything on the internet. (Who else loves Nintendo?) so I haven't had this container spun up in awhile. I haven't even played since launch, but I paid for the game and set up a server just cause.

When I googled "Palworld server github", I laughed cause the first result was the same dev as the sons server I run. I thought it was gonna be hard but they made this one simple, just follow his README.

https://github.com/jammsen/docker-palworld-dedicated-server

I run this container on the same VM as Sons, limiting IP reservations as well as vulnerable systems.

Same thing goes for folder structure here, I just made a pal folder in home directory. I do the same thing with port forwards as I do for Sons

Thanks to the Developers on these repo's for your work.