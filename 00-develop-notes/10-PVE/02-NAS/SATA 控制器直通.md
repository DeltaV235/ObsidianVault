---
title: SATA 控制器直通
created: 2024-12-19
tags:
    - PVE
    - Passthrough
    - NAS
---

# SATA 控制器直通

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

## 3.更新 grub 引导以及 ramfs

- 更新 grub 引导

 ```bash
 update-grub
 ```

- 更新 ramfs

```bash
update-initramfs -u -k all
```

## 4.检查 IOMMU 是否正常

如若正常，所有设备均有独立 `IOMMU Group` 编号，重启后使用 `dmesg | grep iommu` 命令查看,如下图；

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
