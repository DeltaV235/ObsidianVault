---
title: 构建 Immortalwrt 镜像
created: 2024-11-29
tags:
    - PVE
    - HomeServer
    - OpenWrt
    - Immortalwrt
---

# 构建 Immortalwrt 镜像

## Ref

设备：Generic x86/64

[firmware selector for Immortlawrt](https://firmware-selector.immortalwrt.org)
[Immortalwrt USTC mirrors](https://mirrors.ustc.edu.cn/immortalwrt/releases/23.05.4/packages/x86_64/luci/?sort=size&order=desc)


## 预装软件包

- 添加
    - luci-app-openclash

## 首次启动时运行的脚本（uci-defaults）

修改 lan ip 地址为内网网段

```bash
# Beware! This script will be in /rom/etc/uci-defaults/ as part of the image.
# Uncomment lines to apply:
#
# wlan_name="ImmortalWrt"
# wlan_password="12345678"
#
# root_password=""
lan_ip_address="192.168.0.21"
#
# pppoe_username=""
# pppoe_password=""
```
