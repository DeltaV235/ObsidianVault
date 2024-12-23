---
title: 通过 Live CD 恢复 PVE
created: 2024-12-22
tags:
    - PVE
    - LiveCD
    - Recovery
---

由于需要直通 AMD 核显，所以更新了 `/etc/default/grub` 和 `/etc/modprobe.d/pve-blacklist.conf`，导致 PVE 无法正常启动，无法访问 webUI。
进入 Recovery Mode 后，PVE 也没有正常启动。

## 恢复方法

通过U盘启动 Live CD，挂载 PVE 的系统盘，修改配置文件。

### 1. 启动 Live CD

- 开机时通过U盘启动，如果是 PVE 的安装盘，选择 `Terminal, Debug mode`。然后 `^d` 退出 debug mode。

### 2. 挂载 PVE 系统盘

检查分区信息，找到 PVE 的 `pve-root`，`EFI` 分区。

```bash
lsblk -f
```

![[Pasted image 20241222193629.png]]

上图中，nvme0n1p2 是 包含 `pve-root` 的 ，nvme0n1p2 是 `EFI` 分区。
