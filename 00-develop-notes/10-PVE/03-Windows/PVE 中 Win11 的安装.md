---
title: PVE 中 Win11 的安装
created: 2024-12-19
tags:
    - PVE
    - Windows
    - Install
    - Tutorial
---

# PVE 中 Win11 的安装

## 挂载 Win11 ISO 和 virtio-win ISO

[virtio-win-archive](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/)

- 在 `Create VM` 时便可设置好上述的两个 CD-ROM。

![[Pasted image 20241219234103.png]]

- System tab 中如下图所示，启用 EFI 分区和 TPM2.0。
![[Pasted image 20241219234128.png]]

- Disk 启用 SSD 模拟。

![[Pasted image 20241219234243.png]]

## 安装 Win11

直接安装无法显示可用的硬盘，需要先在挂载的 `virtio-win` 中安装 `vioscsi/w11` 驱动。

Win11 安装完成后，运行 `virtio-win` 中的 `virtio-win-guest-tools.exe` 安装驱动。
