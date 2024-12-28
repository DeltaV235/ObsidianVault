---
title: SATA 控制器直通
created: 2024-12-19
tags:
    - PVE
    - Passthrough
    - NAS
    - Tutorial
---

## 1.编辑 `/etc/modules`

加入如下内容：

```bash
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

## 2.编辑 `/etc/default/grub`

找到 `GRUB_CMDLINE_LINUX_DEFAULT="quiet"`,修改为：

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on pcie_acs_override=downstream,multifunction"
```

Intel 的 CPU 就改成：

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on pcie_acs_override=downstream,multifunction"
```

## 3.禁用 ahci 驱动

编辑 `/etc/modprobe.d/blacklist.conf`，添加如下内容：

```bash
blacklist ahci
```

若不禁用 `ahci` 驱动，SATA Controller 在 PVE 启动后会被 PVE 系统占用，所有硬盘保持运行状态。此时在 PVE node 的 Disk 页面中可以查看到硬盘信息。

当直通的 VM 启动后，PVE 需要将 SATA Controller 移交给 VM，PVE 会卸载 ahci 模块，此时所有在 SATA Controller 上的硬盘都会停转。

当 VM 拿到 SATA Controller 后，硬盘又会重新起转。

VM 关机后 SATA Controller 并不会交还给 PVE。此时硬盘处于停转状态。

综上所述，若不禁用 `ahci` 驱动，硬盘会在 PVE 启用时启转一次，VM 启动时停转一次并再启转一次。最终导致硬盘启停次数/断电磁头缩回计数/磁头加载卸载循环计数 +2。

禁用了 `ahci` 驱动后，硬盘在 PVE 启动时依然会启转，但在 VM 启动时不会停转，一直保持运行状态。只有在 VM 关机后硬盘才会停转。在正常 PVE 和 VM 开机的场景下，硬盘启停次数/断电磁头缩回计数/磁头加载卸载循环计数只增加一次。

## 4.更新 grub 引导以及 ramfs

- 更新 grub 引导

 ```bash
 update-grub
 ```

- 更新 ramfs

```bash
update-initramfs -u -k all
```

- 重启

## 5.检查 IOMMU 是否正常

如若正常，所有设备应该均有独立 `IOMMU Group` 编号，重启后使用 `dmesg | grep iommu` 命令查看,如下图；

![[Pasted image 20241219154427.png]]

```bash
lspci
```

![[Pasted image 20241219154502.png]]

如果你有多块硬盘，且不知道硬盘属于哪个控制器，你可以通过下面命令查看。

```bash
ls -la /sys/dev/block/|grep -v loop |grep -v dm
```

在 `Raw Device` 中 `SATA Controller` 应该独自占用一个 `IOMMU Group`。无需勾选 `All Functions`，因为这样会导致 `16:00.0` 被选中。

![[Pasted image 20241219154554.png]]

## Reference

[关于自建虚拟机下NAS的硬盘直通，你姿势对了吗？](https://www.bilibili.com/opus/872260149328216071)
