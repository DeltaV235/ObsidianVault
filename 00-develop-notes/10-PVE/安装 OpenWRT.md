---
title: 安装 OpenWRT
created: 2024-11-27
tags:
    - PVE
    - HomeServer
    - OpenWRT
    - SoftRouter
---

# 安装 OpenWRT

PVE 虚拟机中不设置 CD/DVD disc image file (iso)，先完成 OpenWRT 虚拟机的创建。随后执行一下命令。

```bash
qm importdisk 100 /var/lib/vz/template/iso/openwrt.img local-lvm
```

**参数说明：**

1. **`qm`**  
    Proxmox 的虚拟机管理工具，用于创建、管理和配置虚拟机。
    
2. **`importdisk`**  
    导入磁盘镜像的子命令，将镜像文件作为虚拟机的新磁盘导入。
    
3. **`100`**  
    虚拟机的 ID。此命令会将镜像导入到 ID 为 100 的虚拟机中。
    - 确保虚拟机 ID 100 已经存在，可以通过 `qm list` 检查已创建的虚拟机。
    - 如果虚拟机尚未创建，可以使用 `qm create 100` 先创建虚拟机。
    
4. **`/var/lib/vz/template/iso/openwrt.img`**  
    镜像文件的路径。这里是一个 OpenWRT 系统的镜像文件，位于 Proxmox 的模板目录下。
    
5. **`local-lvm`**  
    存储目标，表示将镜像导入到 Proxmox 存储池 `local-lvm` 中。
    
    - `local-lvm` 是默认的存储池，通常用于存储磁盘映像文件。如果有其他存储目标（如 `ceph` 或 `zfs`），可以替换为相应的存储名称。

 **命令作用**

- 将指定的磁盘镜像文件（`openwrt.img`）导入到虚拟机 ID 为 `100` 的虚拟机，并将其存储到 `local-lvm` 中。
