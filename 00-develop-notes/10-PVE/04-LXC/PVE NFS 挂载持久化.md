---
title: PVE NFS 挂载持久化
created: 2025-01-04
tags:
    - Linux
    - NFS
    - LXC
    - fstab
---

```bash
192.168.0.22:/mnt/Prod-01/Media /mnt/nas/media nfs rw,relatime,vers=4.2,rsize=1048576,wsize=1048576,hard,proto=tcp,timeo=600,retrans=2,sec=sys,bg,x-systemd.automount 0 0
192.168.0.22:/mnt/Prod-01/AppData /mnt/nas/appdata nfs rw,relatime,vers=4.2,rsize=1048576,wsize=1048576,hard,proto=tcp,timeo=600,retrans=2,sec=sys,bg,x-systemd.automount 0 0
```
