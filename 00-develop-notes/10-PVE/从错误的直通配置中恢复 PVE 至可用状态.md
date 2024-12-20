---
title: 从错误的直通配置中恢复 PVE 至可用状态
created: 2024-12-19
tags:
    - PVE
    - Passthrough
---

# 从错误的直通配置中恢复 PVE 至可用状态

如果目标直通设备和其他设备在同一个 IOMMU 组中，会导致将整个 IOMMU 组直通给虚拟机。
如果 IOMMU 组中有 `USB3.1 Controller` 或者 `RTL8125 2.5GbE Controller` 等设备，会导致 PVE 和所有在 PVE 上的 VM 无法通过网络进行访问，宿主机也无法通过键盘进行输入。

## 解决方法

1. 点按机箱 Power Button 使 PVE 正常关机后，再开机，进入 GRUB 引导界面
2. 进入 `Advanced options for Proxmox VE`，选择 `Proxmox VE, with Linux 5.15.0-9-pve (recovery mode)`
3. 启用 WebUI 服务 `systemctl start pveproxy`
4. 进入 WebUI，重新配置直通设备或者删除错误的直通配置
5. 重启 PVE，恢复正常使用
