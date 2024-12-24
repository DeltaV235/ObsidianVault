---
title: OpenWrt 配置
created: 2024-11-28
tags:
    - OpenWrt
    - PVE
    - HomeServer
    - DHCP
    - Tutorial
---

# OpenWrt 配置

## Interface

修改 Gateway/DNS 为主路由 IP，使 `OpenWrt` 能够访问外网。

![[Pasted image 20241128171717.png]]

![[Pasted image 20241128172212.png]]

如果 OpenWrt 作为旁路由，将设备设置为 `eth0`

![[Pasted image 20241202233532.png]]

如果不使用 IPv6，在高级设置中，禁用 `委托 IPv6 前缀` 和 `IPv6 分配长度`。

![[Pasted image 20241202233636.png]]

## DHCP

### 主路由

在主路由上关闭或使用自动 DHCP。自动”状态为在使用WDS功能时，路由器将检测局域网中是否有别的DHCP服务器存在，若存在则会关闭自身的DHCP服务器。

并确保主路由 DHCP Range 和软路由的 DHCP Range 不重叠。

![[Pasted image 20241128171412.png]]

**Main Router DHCP Range: 192.168.0.80 - 192.168.0.159**

### OpenWrt

确保主路由 DHCP Range 和软路由的 DHCP Range 不重叠。

**OpenWrt DHCP Range: 192.168.0.160 - 192.168.0.239**

![[Pasted image 20241128172318.png]]

启用高级设置中的强制选项

![[Pasted image 20241128172551.png]]

## DHCP/DNS

来到网络中的 DHCP/DNS 页面，启用唯一授权。开启后 `/ect/config/dhcp` 中的 `config dnsmasq` 会新增 `option authoritative '1'`。`authoritative` 设置告诉 dnsmasq 它是网络中的主要 DHCP 服务器，优先响应所有 DHCP 请求。这在网络中有多个 DHCP 服务器时可以提高 dnsmasq 的优先级。

![[Pasted image 20241128172758.png]]

![[Pasted image 20241128173133.png]]

### 扫描网络中的 DHCP Server

```bash
sudo nmap --script broadcast-dhcp-discover
```

```bash
sudo nmap --script broadcast-dhcp-discover
Password:
Starting Nmap 7.95 ( https://nmap.org ) at 2024-11-28 17:32 CST
NSOCK ERROR [0.0770s] nsock_pcap_open(): pcap_activate(anpi2) WARNING: BIOCPROMISC: Operation not supported on socket.
NSOCK ERROR [0.0790s] nsock_pcap_open(): pcap_activate(anpi0) WARNING: BIOCPROMISC: Operation not supported on socket.
NSOCK ERROR [0.0790s] nsock_pcap_open(): pcap_activate(anpi1) WARNING: BIOCPROMISC: Operation not supported on socket.
Pre-scan script results:
| broadcast-dhcp-discover:
|   Response 1 of 1:
|     Interface: en0
|     IP Offered: 192.168.0.249
|     DHCP Message Type: DHCPOFFER
|     Server Identifier: 192.168.0.21
|     IP Address Lease Time: 12h00m00s
|     Renewal Time Value: 6h00m00s
|     Rebinding Time Value: 10h30m00s
|     Subnet Mask: 255.255.255.0
|     Broadcast Address: 192.168.0.255
|     Router: 192.168.0.21
|     Domain Name Server: 192.168.0.21
|_    Domain Name: lan
WARNING: No targets were specified, so 0 hosts scanned.
Nmap done: 0 IP addresses (0 hosts up) scanned in 10.08 seconds
```

![[Pasted image 20241128175212.png]]

### 静态地址分配

将需要 Proxy 的设备的 Mac 与 IP 绑定，并设置 tag 为 proxy。

![[Pasted image 20241128221736.png]]

![[Pasted image 20241128224551.png]]

### 设置  DHCP

将所有没有被打上 proxy tag 的设备的网管和 DNS 都设置为主路由 IP。
在该 OpenWrt 旁路网管下线的情况下，直连设备不会受到影响，并且在续租时将被主路由 DHCP Server 分配 IP。
对于走 proxy 的设备，由于 OpenWrt 作为其网管，下线后将无法访问外网，直到设备 IP 租期到期，续租时将被主路由分配新的网管和 DNS，即主路由 IP。这时 proxy 设备才能正常访问外网。
所以对于配置了 DHCP IP/MAC 绑定的 proxy 设备，租期并没有设置为 infinite。在 OpenWrt 不可用的情况下，能在最长为租期的时间之后，自行恢复外网访问。

![[OpenWRT 通过配置 DHCP 服务为不同设备分配不同的 Gateway 和 DNS IP#代理白名单]]

![[Pasted image 20241128222106.png]]

![[Pasted image 20241128224514.png]]

## Firewall

禁用 `SYN-Flood Protection` 和 `FullCone NAT6`。
如果 OpenWrt 作为旁路网管，`Zone` 只保留 `lan`。

![[Pasted image 20241202234614.png]]
