+++
date = 2024-02-06T02:58:07Z
description = ""
draft = false
slug = "resize-ubuntu-vm-disk"
title = "Resize Ubuntu VM Disk in Proxmox"
tags = ["Ubuntu"]
showToc = false
+++


# 1st step: Increase/resize disk from GUI console

![Proxmox webui change](/images/prox-resize.png#center)

# 2nd step: Extend physical drive partition and check free space with:

```bash
sudo growpart /dev/sda 3
```

```bash
sudo pvdisplay
```

```bash
sudo pvresize /dev/sda3
```

```bash
sudo pvdisplay
```



# 3rd step: Extend Logical volume



```bash
sudo lvdisplay
```

```bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

```bash
sudo lvdisplay
```

# 4th step: Resize Filesystem

```bash
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```

```bash
sudo fdisk -l
```



