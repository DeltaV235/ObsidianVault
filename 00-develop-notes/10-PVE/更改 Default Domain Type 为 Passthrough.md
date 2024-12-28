---
title: 更改 Default Domain Type 为 Passthrough
created: 2024-12-27
tags:
    - PVE
    - Passthrough
---

根据 [PVE 文档 - PCI(e) Passthrough](https://pve.proxmox.com/wiki/PCI(e)_Passthrough#:~:text=If%20your%20hardware%20supports%20IOMMU%20passthrough%20mode%2C%20enabling%20this%20mode%20mightincrease%20performance.) 中的描述

> If your hardware supports IOMMU passthrough mode, enabling this mode might increase performance. This is because VMs then bypass the (default) DMA translation normally performed by the hyper-visor and instead pass DMA requests directly to the hardware IOMMU.

可以通过修改 `Default Domain Type` 为 `Passthrough` 来提高性能。

## 修改 GRUB 配置

所以在 `/etc/default/grub` 中添加了 `iommu=pt` 参数，如下所示：

```bash
vi /etc/default/grub
```

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet initcall_blacklist=sysfb_init iommu=pt pcie_acs_override=downstream,multifunction"
```

更新 GRUB 配置并重启

```bash
update-grub

reboot
```

添加前 Default Domain Type 为 `translated`，添加后为 `passthrough`。

![[屏幕截图 2024-12-27 133324.png]]

![[屏幕截图 2024-12-27 124113.png]]

## 性能测试

**Default Domain Type = Translated:**

![[FireStrike-2024-12-27-134459-Translate.png]]

![[TimeSpy-2024-12-27-122518-Translate.png]]

**Default Domain Type = Passthrough:**

![[FireStrike-2024-12-27-132541-PT.png]]

![[TimeSpy-2024-12-27-125043-PT.png]]

实际性能并没有提升。
