---
title: LXC 挂载宿主机 NFS
created: 2025-01-06
language: Bash
tags:
    - PVE
    - LXC
    - NFS
---

修改 `/etc/pve/lxc/<CTID>.conf`，添加如下挂载点

```bash
mp0: /mnt/nas/appdata/docker,mp=/root/appdata
mp1: /mnt/nas/media,mp=/root/archive
```

^4f5f0d

```bash
mp0: /mnt/nas/appdata/docker,mp=/root/nas-app-data-docker
```
