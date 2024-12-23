---
date: 2023-08-25T00:13:40Z
description: ""
draft: false
slug: "hardware"
tags: 
  - "Homelab"
title: "Hardware"
---

Most of my infrastructure is hosted on my in-lab Proxmox server, along with a few new machines for dedicated services. Here are some of the specs of some of the in-lab machines.

## Proxmox Node 1

- CPU: Intel Core i7-9700K
- GPU: Nvidia GeForce GTX 1060 6GB
- RAM: 64GB DDR4 3000Mhz
- NVME SSD's for storage
- 4x 4TB HDD's (passthrough to NAS)

## Proxmox Node 2

- CPU: Intel Core i7-8700K
- GPU: Nvidia GeForce GTX 1660 6GB
- RAM: 64GB DDR4
- NVME SSD's for storage


## Gaming PC 

![Image Description](/images/Pasted%20image%2020241222231541.png)

- CPU: Intel Core i7-13700K
- GPU: Nvidia GeForce RTX 3080
- RAM: 64GB DDR5 6000 Mhz
- SSD: Samsung 980 Pro 2TB
- Mobo: [MPG Z790 EDGE WIFI](https://www.msi.com/Motherboard/MPG-Z790-EDGE-WIFI)
- Windows 11 Pro

Main PC used for everything. I just remote into every other machine. Yes, it is on top of my mini-fridge. Yes, my cable management is terrible.

## Networking

![Image Description](/images/Pasted%20image%2020241223012544.png)
- ISP: Xfinity. Coax currently getting 2.0Gbps download and 350mbps upload. (my monitoring in lab averages 2.21Gbps down and 360mbps up)
- Router: [pfSense Box](https://a.co/d/9JgFmt6)
- AP's: TP-Link Deco XE75 PRO (x3) WIFI 6E Mesh
- Switch: TRENDnet 6-port 10G