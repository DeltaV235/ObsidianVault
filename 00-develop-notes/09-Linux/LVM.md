---
title: LVM
created: 2024-11-27
tags:
    - Linux
    - Filesystem
---

# LVM

## 介绍

LVM（Logical Volume Manager）是一个逻辑卷管理器，它允许用户管理磁盘空间，而不用关心物理磁盘的细节。LVM 通过将物理磁盘划分为一个或多个物理卷（Physical Volume，PV），然后将物理卷组合成一个或多个卷组（Volume Group，VG），最后将卷组划分为一个或多个逻辑卷（Logical Volume，LV）来实现这一目的。

## 基本概念

![[Pasted image 20241127101649.png]]

- 物理卷（Physical Volume，PV）：物理卷是一个物理磁盘或分区，它是 LVM 的最底层。物理卷上的数据是线性排列的，没有任何逻辑结构。
- 卷组（Volume Group，VG）：卷组是一个或多个物理卷的集合，它是 LVM 的中间层。卷组上的数据是条带化的，可以跨越多个物理卷。
- 逻辑卷（Logical Volume，LV）：逻辑卷是卷组的一部分，它是 LVM 的最顶层。逻辑卷上的数据是线性排列的，没有任何逻辑结构。

## 常用命令

- `pvcreate`：创建物理卷。
- `vgcreate`：创建卷组。
- `lvcreate`：创建逻辑卷。
- `pvdisplay`：显示物理卷的信息。
- `vgdisplay`：显示卷组的信息。
- `lvdisplay`：显示逻辑卷的信息。
- `pvmove`：移动物理卷上的数据。
- `vgextend`：扩展卷组。
- `lvextend`：扩展逻辑卷。
- `pvremove`：删除物理卷。
- `vgremove`：删除卷组。
- `lvremove`：删除逻辑卷。

## 示例

### 创建物理卷

在创建物理卷之前，需要先创建一个分区。

```bash
# 创建一个 10G 的分区
fdisk /dev/sdb
```

```bash
pvcreate /dev/sdb1
```

### 创建卷组

```bash
vgcreate vg0 /dev/sdb1
```

### 创建逻辑卷

```bash
# 创建一个 10G 的逻辑卷 lv0 并将其添加到卷组 vg0 中
lvcreate -L 10G -n lv0 vg0
```

### 格式化逻辑卷

```bash
mkfs.ext4 /dev/vg0/lv0
```

### 挂载逻辑卷

```bash
mount /dev/vg0/lv0 /mnt
```

### 扩展逻辑卷

```bash
# 扩展逻辑卷 lv0 到 20G
lvextend -L 20G /dev/vg0/lv0
```

逻辑卷扩展完成后，还需要扩展文件系统

```bash
resize2fs /dev/vg0/lv0
```

### 缩小逻辑卷

在缩小逻辑卷之前，需要先缩小文件系统

```bash
resize2fs /dev/vg0/lv0 10G
```

```bash
# 缩小逻辑卷 lv0 到 10G
lvreduce -L 10G /dev/vg0/lv0
```

### 移动物理卷上的数据

```bash
# 将物理卷 /dev/sdb1 上的数据移动到 /dev/sdc1
pvmove /dev/sdb1 /dev/sdc1
```

### 删除逻辑卷

```bash
lvremove /dev/vg0/lv0
```

### 删除卷组

```bash
vgremove vg0
```

### 删除物理卷

```bash
pvremove /dev/sdb1
```

## 参考链接

- [Linux LVM简明教程](https://linux.cn/article-3218-1.html)
- [LVM管理](https://www.cnblogs.com/diantong/p/10554831.html)
- [Logical Volume Manager - ArchWiki](https://wiki.archlinux.org/title/LVM)
- [Logical Volume Manager - Wikipedia](https://en.wikipedia.org/wiki/Logical_Volume_Manager)
