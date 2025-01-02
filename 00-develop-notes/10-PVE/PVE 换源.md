---
title: PVE 换源
created: 2024-12-19
tags:
    - PVE
    - APT
    - Source
    - Tutorial
---

## 关闭企业源和ceph源

- `/etc/apt/sources.list.d/ceph.list` 注释以下内容

```bash
#deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise
```

- `/etc/apt/sources.list.d/pve-enterprise.list` 注释以下内容

```bash
#deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise
```

## 更换 APT 为 USTC 镜像源

[USTC-Debian-Mirrors](https://mirrors.ustc.edu.cn/help/debian.html)

替换 `/etc/apt/sources.list` 为如下

```bash
# 默认注释了源码仓库，如有需要可自行取消注释
deb http://mirrors.ustc.edu.cn/debian bookworm main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian bookworm main contrib non-free non-free-firmware
deb http://mirrors.ustc.edu.cn/debian bookworm-updates main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian bookworm-updates main contrib non-free non-free-firmware

# backports 软件源，请按需启用
# deb http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-firmware

# security updates
deb http://security.debian.org bookworm-security main contrib
```

## 更换 CT Template(LXC) 为 tsinghua 镜像源

- 备份 `/usr/share/perl5/PVE/APLInfo.pm`

```bash
cp /usr/share/perl5/PVE/APLInfo.pm /usr/share/perl5/PVE/APLInfo.pm.bak
```

- 替换 `/usr/share/perl5/PVE/APLInfo.pm` 中的 `http://download.proxmox.com` 为 `https://mirrors.tuna.tsinghua.edu.cn/proxmox`

```bash
sed -i 's|http://download.proxmox.com|https://mirrors.tuna.tsinghua.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm
```

- 重启 pvedaemon

```bash
systemctl restart pvedaemon.service
```
