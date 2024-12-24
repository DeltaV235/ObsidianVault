---
title: 通过 Live CD 恢复 PVE
created: 2024-12-22
tags:
    - PVE
    - LiveCD
    - Recovery
    - Tutorial
---

由于需要直通 AMD 核显，所以更新了 `/etc/default/grub` 和 `/etc/modprobe.d/pve-blacklist.conf`，导致 PVE 无法正常启动，无法访问 webUI。
进入 Recovery Mode 后，PVE 也没有正常启动。

## 恢复方法

通过U盘启动 Live CD，挂载 PVE 的系统盘，修改配置文件。

### 1. 启动 Live CD

- 开机时通过U盘启动，如果是 PVE 的安装盘，选择 `Terminal, Debug mode`。然后 `^d` 退出 debug mode。

### 2. 挂载 PVE 系统盘

**检查分区信息，找到 PVE 的 `pve-root`，`EFI` 分区。**

```bash
lsblk -f
lsblk
```

![[Pasted image 20241222193629.png]]

![[Pasted image 20241223154831.png]]

上图中，nvme0n1p3 是 包含 `pve-root` 的 PV，nvme0n1p2 是 `EFI` 分区。

**挂载 `pve-root` 和 `EFI` 分区。**

```bash
mount /dev/mapper/pve-root /mnt
mount /dev/nvme0n1p2 /mnt/boot/efi
```

`pve-root` 分区挂载到 `/mnt` 后，`/mnt` 下的文件即为 PVE 的根目录。
`EFI` 分区挂载到 `/mnt/boot/efi` 后，目录结构为 `/mnt/boot/efi/EFI`。

**挂载 `/proc`, `/sys`, `/dev`。**

```bash
mount -t proc /proc /mnt/proc
mount --rbind /sys /mnt/sys
mount --rbind /dev /mnt/dev
```

#### **总结的挂载命令清单**

|类型|命令|
|---|---|
|挂载根分区|`mount /dev/mapper/pve-root /mnt`|
|挂载 EFI 分区|`mount /dev/nvme0n1p1 /mnt/boot/efi`|
|挂载数据分区 `/var/lib/vz`|`mount /dev/mapper/pve-data /mnt/var/lib/vz`|
|挂载 `/proc`|`mount -t proc /proc /mnt/proc`|
|挂载 `/sys`|`mount --rbind /sys /mnt/sys`|
|挂载 `/dev`|`mount --rbind /dev /mnt/dev`|
|验证挂载|`df -h` 或 `lsblk -f`|
|卸载分区|`umount /boot/efi`|

### 3. 修改配置文件

修正 `/etc/default/grub` 和 `/etc/modprobe.d/pve-blacklist.conf`。

```bash
vi /mnt/etc/default/grub
vi /mnt/etc/modprobe.d/pve-blacklist.conf
```

### 4. 更改根目录

```bash
chroot /mnt
```

> 通过 chroot，可以将一个进程及其子进程的根目录更改为指定的目录，从而创建一个“受限的虚拟环境”或“伪根环境”。

### 5. 更新 grub

```bash
update-grub
```

### 6. 更新 initramfs

```bash
update-initramfs -u -k all  # 更新所有内核
```

检测 `/boot` 下的文件是否更新。

`initrd.img-*-pve` 的时间戳应该是最新的。

### 7. 重启

`ctrl + alt + del` 重启设备。
