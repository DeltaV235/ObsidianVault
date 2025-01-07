---
title: Proxmox VE 非特权 LXC 容器挂载 NFS
created: 2025-01-06
tags:
    - PVE
    - LXC
    - NFS
---

由于非特权 LXC 容器无法挂载 NFS，特权 LXC 容器又不够安全，因此需要一种方法在非特权 LXC 容器中挂载 NFS。

## Reference

[在特权/无特权LXC容器中挂载SMB/NFS/WebDAV等NAS服务](https://post.smzdm.com/p/al82p3eg/)
[pve安装docker好去处，再谈lxc安装docker，顺带安装docker中文管理界面](https://post.smzdm.com/p/a30km0k5/)
[Proxmox VE 创建自定义的 LXC 容器 CT 模板](https://blog.hellowood.dev/posts/proxmox-ve-创建自定义的-lxc-容器-ct-模板/)

## 解决方法

### 1. 在 Proxmox VE 主机上挂载 NFS

首先，在 Proxmox VE 主机上挂载 NFS，然后将挂载点绑定到非特权 LXC 容器中。

根据 [[ZFS Dataset NFS 挂载问题#最终解决方案|ZFS Dataset NFS 挂载问题的最终解决方案]] ，只将父 dataset 挂载到 PVE 上，并持久化。

![[PVE NFS 挂载持久化#^80ff5a]]

### 2. 测试 fstab

在 Proxmox VE 主机上测试 fstab 是否正确

```bash
mount -av
```

检查是否有错误信息，并使用 `df -h` 检查挂载点是否正确。

### 3. 在 LXC 容器中绑定挂载点

#### 3.1 使用 `pct` 命令挂载

在非特权 LXC 容器中，使用 `pct` 命令将 Proxmox VE 主机上的挂载点绑定到容器中。

```bash
pct set <CTID> --mp1 /mnt/nas/appdata,mp=/root/appdata
pct set <CTID> --mp0 /mnt/nas/media,mp=/root/archive
```

#### 3.2 修改配置文件

在非特权 LXC 容器中，修改配置文件 `/etc/pve/lxc/<CTID>.conf`，添加以下内容：

![[LXC 挂载宿主机 NFS#^4f5f0d]]
