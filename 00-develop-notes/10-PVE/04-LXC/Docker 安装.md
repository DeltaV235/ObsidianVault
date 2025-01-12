---
title: Docker 安装
created: 2025-01-06
language: Bash
tags:
    - Docker
---

## Ubuntu

```bash
apt install docker.io
```

## Alpine

### 1. 更新系统和包管理器

确保 Alpine 的包管理器 `apk` 已更新：

```bash
apk update && apk upgrade
```

---

### 2. 安装 Docker

Docker 包已经包含在 Alpine 的官方包管理器中，无需额外添加源。直接运行以下命令安装 Docker：

```bash
apk add docker
```

---

### 3. 启动 Docker 服务

安装完成后，需要启动 Docker 服务，并将其设置为开机自启：

```bash
# 启动 Docker 服务
rc-service docker start

# 设置 Docker 服务开机自启
rc-update add docker default
```

---

### 4. 验证 Docker 安装

运行以下命令检查 Docker 是否成功安装并启动：

```bash
docker --version
```

输出类似如下，表示安装成功：

```
Docker version 20.10.17, build 100c701
```

然后运行以下命令验证 Docker 服务是否运行正常：

```bash
docker info
```
