---

title: Zerotier 搭建 VPN
created: 2025-05-04
tags:
    - VPN
    - Zerotire

---

## Zerotier Client

### 安装 Zerotier

```bash
curl -s https://install.zerotier.com/ | bash
```

或者使用 `apt` 安装

```bash
apt install zerotier-one
```

### 启动 Zerotier 服务

```bash
systemctl start zerotier-one
```

设置开机自启动

```bash
systemctl enable zerotier-one
```

### 加入 Zerotier 网络

在 Zerotier 官网注册账号，并创建一个网络

[Zerotier Central](https://my.zerotier.com/)

```bash
zerotier-cli join <network_id>
```

### 查看 Zerotier 网络

```bash
zerotier-cli listnetworks
```

### 查看 Zerotier 节点状态

```bash
zerotier-cli listpeers
```

```bash
zerotier-cli peers
```

### OpenWrt 配置

在 `OpenWrt` 中使用 `Zerotier`，还需要安装对应的 `luci` 插件。并启用 `Auto NAT Clients`，使所有以 OpenWrt 为网关的设备都可以通过 Zerotier 网络访问外网设备，或被外网设备访问。

![[Pasted image 20250504224456.png]]

如果使用的是 OpenWrt 系统，需要添加 Interface。

在 `Network` -> `Interfaces` 中添加 `Zerotier` 接口，其中的 `Device` 选择 `zerotier` 创建的虚拟网卡。

![[Pasted image 20250504224041.png]]

### 配置 Zerotier 网络的静态路由

为了使外部的设备可以通过内网 IP 访问到 Zerotier 网络中的设备，需要配置 Zerotier 网络的静态路由。

`192.168.0.0/24` -> `192.168.192.xxx(虚拟 IP)`

## Moon

### 安装 Moon

先安装 ZeroTier 客户端

```bash
curl -s https://install.zerotier.com/ | bash
```

### 加入到 Zerotier 网络

```bash
zerotier-cli join <network_id>
```

### 生成 moon.json 配置文件

```bash
cd /var/lib/zerotier-one
sudo zerotier-idtool initmoon identity.public > moon.json
```

### 编辑 moon.json 配置文件

```json
"stableEndpoints": ["1.2.3.4/9993"]  # 替换为 Moon 服务器 IP
```

### 生成 Moon 签名文件

```bash
sudo zerotier-idtool genmoon moon.json
```

### 移动签名文件

```bash
sudo cp /var/lib/zerotier-one/0000000000000000.moon /var/lib/zerotier-one/moons.d/
```

### 重启 ZeroTier 服务并设置开机启动

```bash
sudo systemctl enable zerotier-one
sudo systemctl restart zerotier-one
```

### 客户端连接 Moon

```bash
sudo zerotier-cli orbit <moon-id> <moon-id> 
```

**验证:**

```bash
zerotier-cli peers
```

如果看到类似以下信息，则表示连接成功

```bash
<ztaddr>   <ver>  <role> <lat> <link>   <lastTX> <lastRX> <path>
1234567890 1.14.2 MOON      11 DIRECT   1463     1450     123.123.123.123/9993
```

## Network Controller

### 安装网络控制器 WebUI

[zero-ui](https://github.com/dec0dOS/zero-ui)

```bash
wget https://raw.githubusercontent.com/dec0dOS/zero-ui/main/docker-compose.yml
```

修改 `compose.yml` 文件中的 `ZU_DEFAULT_USERNAME` 和 `ZU_DEFAULT_PASSWORD` 为你的用户名和密码

```bash
ZU_DEFAULT_USERNAME=admin
ZU_DEFAULT_PASSWORD=admin
```

将 `ZU_SECURE_HEADERS` 设置为 `false` 以禁用 HTTPS

```bash
ZU_SECURE_HEADERS=false
```

将 `/var/lib/zerotier-one/` 目录挂载到容器中

```bash
volumes:
  - /var/lib/zerotier-one:/var/lib/zerotier-one
```

删除另外两个容器 `zerotier-one` 和 `Caddy`，因为 `zerotier-one` 已经在物理机上运行了

启动容器

```bash
sudo docker compose up -d && sudo docker compose logs -f
```

允许服务器的 tcp `4000` 端口和 tcp/udp `9993` 端口。

访问 `http://<ip>:4000` (默认端口号为 `4000`) 即可看到 Network Controller 的 WebUI

### 拒绝 4000 端口

在 Network Controller 加入到 Zerotier 网络后，拒绝 4000 端口，后续所以访问通过虚拟 IP 访问。不再允许通过公网访问。
