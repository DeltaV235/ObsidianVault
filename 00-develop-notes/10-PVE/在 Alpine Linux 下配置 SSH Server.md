---
title: 在 Alpine Linux 下配置 SSH Server
created: 2025-11-13
tags:
    - Alpine
    - SSH
    - Linux
    - LXC
---

## 概述

Alpine Linux 支持两种 SSH 服务器实现:
- **OpenSSH**: 功能完整的标准 SSH 服务器
- **Dropbear**: 轻量级的 SSH 服务器替代方案,适合资源受限环境

本文档主要介绍 OpenSSH 的安装和配置方法。

## OpenSSH 安装与启动

### 1. 安装 OpenSSH

```bash
apk add openssh
```

### 2. 配置服务自启动

```bash
# 添加到开机自启
rc-update add sshd

# 立即启动服务(同时生成配置文件)
rc-service sshd start
```

## SSH 服务器配置优化

### 配置文件位置

主配置文件: `/etc/ssh/sshd_config`

### 常用优化配置

编辑配置文件以提升性能和安全性:

```bash
vi /etc/ssh/sshd_config
```

**推荐配置项:**

```conf
# 禁用 DNS 反向解析,加快连接速度
UseDNS no

# 禁用密码认证,强制使用密钥登录(可选,根据需求)
PasswordAuthentication no
```

### 应用配置更改

```bash
# 重启 SSH 服务使配置生效
rc-service sshd restart
```

## 端口配置

### 默认端口

SSH 服务默认监听 **TCP 22 端口**

### 修改监听端口

**1. 检查目标端口是否被占用:**

```bash
netstat -lnp | grep <目标端口>
```

**2. 修改配置文件:**

```conf
# 在 /etc/ssh/sshd_config 中修改
Port <新端口号>
```

**3. 重启服务:**

```bash
rc-service sshd restart
```

## 密钥认证配置

### 生成 SSH 密钥对(在客户端)

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### 将公钥复制到 Alpine 服务器

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@alpine-server
```

或手动添加到服务器:

```bash
# 在 Alpine 服务器上
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "你的公钥内容" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

## Dropbear 轻量级替代方案

### 安装 Dropbear

```bash
apk add dropbear
```

### 配置文件位置

`/etc/conf.d/dropbear`

### 客户端工具

Dropbear 客户端需要单独安装:

```bash
apk add dropbear-dbclient
```

### 服务管理

```bash
# 启动服务
rc-service dropbear start

# 添加到开机自启
rc-update add dropbear
```

## RAM 环境持久化配置

如果 Alpine 运行在 RAM 环境(如 diskless 模式),需要保存配置:

```bash
# 保存当前配置到存储介质
lbu ci
```

**说明:**
- `lbu` (Local Backup Utility) 用于在 diskless 模式下持久化配置
- 每次修改配置后都需要执行此命令

## 防火墙配置

如果启用了防火墙,需要开放 SSH 端口:

```bash
# 使用 iptables
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 或使用 awall (Alpine Wall)
apk add awall
# 编辑防火墙规则允许 SSH
```

## 故障排查

### 查看服务状态

```bash
rc-service sshd status
```

### 查看监听端口

```bash
netstat -lntp | grep sshd
```

### 查看日志

```bash
# 查看系统日志
cat /var/log/messages | grep sshd

# 实时监控日志
tail -f /var/log/messages
```

### 测试配置文件语法

```bash
sshd -t
```

## 安全建议

1. **禁用 root 远程登录** (在 `/etc/ssh/sshd_config`):
   ```conf
   PermitRootLogin no
   ```

2. **限制登录用户**:
   ```conf
   AllowUsers user1 user2
   ```

3. **使用密钥认证** 而不是密码认证

4. **修改默认端口** 减少自动化扫描攻击

5. **配置 fail2ban** 防止暴力破解:
   ```bash
   apk add fail2ban
   rc-update add fail2ban
   rc-service fail2ban start
   ```

## 参考链接

- [Alpine Linux Wiki: Setting up a SSH server](https://wiki.alpinelinux.org/wiki/Setting_up_a_SSH_server)
