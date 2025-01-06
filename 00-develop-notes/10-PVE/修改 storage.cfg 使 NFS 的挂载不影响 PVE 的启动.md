---
title: 修改 storage.cfg 使 NFS 的挂载不影响 PVE 的启动
created: 2025-01-05
language: Bash
tags:
    - NFS
    - PVE
---

```bash
nfs: prod-01-vm-backup
        export /mnt/Prod-01/VM-Backup
        path /mnt/pve/prod-01-vm-backup                                 
        server 192.168.0.22
        content backup
        nodes pve-01
        prune-backups keep-all=1
        options bg,hard
```