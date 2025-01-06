---
title: LXC 挂载宿主机 NFS
created: 2025-01-06
language: Java
tags:
---

修改 `/etc/pve/lxc/<CTID>.conf`，添加如下挂载点

```bash
mp0: /mnt/nas/appdata/docker,mp=/root/appdata 
mp1: /mnt/nas/media/,mp=/root/archive/media
```