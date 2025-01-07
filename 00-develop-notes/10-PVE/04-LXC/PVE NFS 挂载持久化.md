---
title: PVE NFS 挂载持久化
created: 2025-01-04
tags:
    - Linux
    - NFS
    - LXC
    - fstab
---

修改 `/etc/fstab`，添加如下内容，其中的 option 与 [[修改 storage.cfg 使 NFS 的挂载不影响 PVE 的启动]] 中的 option 有相同的作用，在 NFS 未上线的情况下，不影响宿主机的启动。

```bash
192.168.0.22:/mnt/Prod-01/Media /mnt/nas/media nfs rw,relatime,vers=4.2,rsize=1048576,wsize=1048576,hard,proto=tcp,timeo=600,retrans=2,sec=sys,bg,x-systemd.automount 0 0
192.168.0.22:/mnt/Prod-01/AppData /mnt/nas/appdata nfs rw,relatime,vers=4.2,rsize=1048576,wsize=1048576,hard,proto=tcp,timeo=600,retrans=2,sec=sys,bg,x-systemd.automount 0 0
```

^80ff5a

**基本挂载选项：**

- ro / rw：
  - ro：以只读模式挂载文件系统。
  - rw：以读写模式挂载文件系统（默认值）。
- hard / soft：
  - hard（默认）：在 NFS 服务器不可用时，I/O 操作会无限重试，直到恢复正常。
  - soft：在 NFS 服务器不可用时，I/O 操作会返回错误，而不是无限重试。适用于非关键数据，但可能导致数据不一致。
- noatime / relatime：
  - noatime：禁止更新文件的访问时间，提升性能。
  - relatime（默认）：仅在文件最近一次访问时间早于修改时间时更新访问时间。
- bg：
  - 如果挂载失败，允许挂载操作在后台重试。

**NFS 特定挂载选项：**

- timeo=\<value>：
  - 指定 NFS 请求超时时间，单位为 1/10 秒。
  - 默认值为 600（即 60 秒）。
  - 较小的值适合不稳定网络，但可能导致频繁超时。
- retrans=\<value>：
  - 指定请求超时后的重试次数，默认值为 2。
- vers=<nfs_version>：
  - 指定使用的 NFS 协议版本：
  - vers=4：使用 NFSv4（默认）。
  - vers=3：使用 NFSv3。
  - vers=2：使用 NFSv2。
