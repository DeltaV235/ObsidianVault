---
title: MacOS 制作 Win11 安装引导盘
created: 2024-11-19
tags:
    - OS
    - Win11
---

# MacOS 制作 Win11 安装引导盘

## Reference

[记一次mac制作win11引导盘并安装](https://juejin.cn/post/7346404753439506482)

## Requirement

1. 下载 Win11 ISO 镜像
2. 8G U盘
3. brew install wimlib 

## 启动盘制作

1. 将U盘的文件系统写入为 `FAT32`
2. 在 macOS 中挂载下载好的 Win11 镜像
3. 通过 `rsync` 复制除了 `sources/install.wim` 之外的所有文件到U盘的根目录

```bash
# CCCOMA_X64FRE_ZH 为iso mount之后的卷名，WIN11 为格式化后的U盘名，每个人可能不一样，可以自己看一下 Volumes 目录下的信息
rsync -vha --exclude=sources/install.wim /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/* /Volumes/WIN11
```

4. 由于 FAT32 单文件最大支持4G，`install.wim` 超过了文件限制，使用 `wimlib` 拆分镜像

```bash
# CCCOMA_X64FRE_ZH 为iso mount之后的卷名，WIN11 为格式化后的U盘名，每个人可能不一样，可以自己看一下 Volumes 目录下的信息，4000 为拆分后最大的文件大小。
wimlib-imagex split /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/sources/install.wim /Volumes/WIN11/sources/install.swm 4000
```

5. 插入U盘至目标机器并安装 Win11