---
title: Raspberry Pi 3B Windows 11 Thin Client 部署指南
created: 2025-12-18
tags:
    - Raspberry
    - Thin-Client
    - Windows
---

# Raspberry Pi 3B Windows 11 Thin Client 部署指南

根据排查与调试过程总结的 **《树莓派 3B 打造 Windows 11 极致瘦客户端（Thin Client）最佳实践指南》**。

这份笔记涵盖了从系统选型、底层驱动修复、RDP 协议调优到网络优化的全链路解决方案。

---

## 1. 核心架构与选型

### 硬件与目标环境
- **硬件设备**: Raspberry Pi 3B (1GB RAM) + 1080p 显示器
- **目标环境**: Windows 11 Pro/Enterprise
- **操作系统**: **Raspberry Pi OS Lite (Legacy, 32-bit)**
  - **原因**: Pi 3B 内存受限，32位系统更节省资源；Bookworm (Legacy) 比 Trixie 更稳定，且 `xserver-xorg` 在旧版驱动下兼容性更佳。

### 软件栈
- **Xorg (X Server)** + **FreeRDP (xfreerdp)** + **Xterm**
- **避坑**: 不安装桌面环境（GNOME/Pixel），不使用 Wayland，不使用 GPU 依赖高的终端（如 Ghostty）。

---

## 2. 为什么选择 Bookworm (Debian 12)？

虽然 Pi 3B 是老硬件，但在连接 Windows 11 这个场景下，**新系统更有优势**。

### 2.1 RDP 协议与 FreeRDP 版本支持

**Windows 11 的要求**：
- Win11 对 RDP 协议的安全性（NLA 认证）和图形压缩算法都有更新

**软件仓库源差异**：
- **Bullseye (旧版, Debian 11)**: 软件源里的 `freerdp2` 版本较旧
- **Bookworm (新版, Debian 12)**: 软件源提供更新的 `freerdp` 库
  - 对 Windows 11 的连接成功率更高
  - 对新版 RDP 图形压缩（H.264/AVC444）的支持更好

**结论**: 为了不出现"协议版本不匹配"或"认证失败"的错误，新版 OS 赢了。

### 2.2 音频架构升级 (PipeWire)

- Bookworm 默认引入了 **PipeWire** 作为音频底层
- 相比老旧的 PulseAudio，PipeWire 在**低延迟**方面表现更好
- **场景收益**: 远程桌面里看视频或听声音时，PipeWire 能稍稍减少"画面动了，声音慢半拍"的现象

---

## 3. 核心限制：必须选 "32-bit Lite"

这是 Pi 3B 运行 Bookworm 的**生死线**。

### 3.1 不要选 64-bit
- 64位系统会增加内存指针的开销
- 对于只有 1GB 内存的 3B，运行 64位 Lite 版虽然能跑
- 但加上 X Server 和 FreeRDP 的视频缓冲区后，**极易触发 OOM (内存溢出)**

### 3.2 不要选 Desktop
- 哪怕是 Bookworm 的 Desktop 版本，也放弃吧
- Desktop 环境会吃掉大量内存

### 3.3 唯一正确选择
**Raspberry Pi OS Lite (32-bit, Bookworm)**
- 这是兼顾新内核与低内存占用的唯一解

---

## 4. Bookworm 部署变化与排雷指南

如果你习惯了旧版树莓派系统，切换到 Bookworm 时，有几个**关键变化**需要注意。

### 4.1 网络配置变化 (dhcpcd → NetworkManager)

**旧版行为**:
- 配置 Wi-Fi 或静态 IP，修改 `/etc/dhcpcd.conf`

**Bookworm 变化**:
- `dhcpcd` 配置文件失效
- 改用 **NetworkManager** 管理网络

**新配置方法**:
```bash
# 方式1: 图形化终端界面（推荐）
sudo nmtui

# 方式2: 命令行工具
sudo nmcli device wifi connect <SSID> password <密码>
```

**运维评价**: NetworkManager 极其稳定，对服务器运维是好事。

### 4.2 X11 vs Wayland 纠葛

**Bookworm Desktop 版默认**:
- 推行 Wayland 作为显示协议

**Lite 版行为**:
- 默认什么图形栈都没有（这正是我们需要的）

**正确安装方式**:
需要手动安装 Xorg 和相关组件，详见**第6章"RDP客户端深度配置"**中的完整安装命令。

**重要说明**:
- 安装完后，Lite 版依然会乖乖使用 **X11 协议**来启动 xfreerdp
- 这很好，因为 FreeRDP 对纯 X11 的硬件加速支持目前比对 XWayland 更成熟稳定
- **不要安装桌面环境**，否则会引入 Wayland 相关依赖

---

## 5. 显示驱动配置（解决黑屏/无输出问题）

### 问题背景
Raspberry Pi OS Bookworm 默认使用 `vc4-kms-v3d` (Full KMS) 驱动。这个驱动对于完整的桌面环境很好，但对于"裸跑" X Server 的 Thin Client 场景，经常会导致 **HDMI 无输出或黑屏**。

### 解决方案：切换到 Fake KMS 模式

编辑 `/boot/firmware/config.txt`：

```bash
sudo nano /boot/firmware/config.txt
```

找到并修改以下配置：

```ini
# 将 Full KMS 改为 Fake KMS（注意多了一个 f）
dtoverlay=vc4-fkms-v3d
max_framebuffers=2

# 调大显存，防止显存不足导致黑屏
gpu_mem=128
```

**配置说明**：
- `vc4-fkms-v3d`: **Fake KMS** 驱动，兼容性更好，适合"裸跑" X Server
- `vc4-kms-v3d`: Full KMS 驱动（Bookworm 默认），会导致黑屏，**需要改掉**
- `max_framebuffers=2`: 允许两个 framebuffer，支持多显示配置
- `gpu_mem=128`: 为 GPU 分配 128MB 内存（Pi 3B 1GB RAM 推荐值）
  - 如果运行 RDP + Xterm 双图形界面，可以调大到 `256`，但会挤占系统内存
  - 建议先用 `128` 测试，如果出现显存不足再调大

**验证 gpu_mem 是否生效**：
```bash
# 查询当前 GPU 内存分配
vcgencmd get_mem gpu
# 输出示例：gpu=128M
```

**替代方案**：
如果 Fake KMS 仍然有问题，可以彻底关闭 3D 加速，使用最原始的 Framebuffer：

```ini
# 注释掉 KMS 驱动（彻底禁用硬件加速）
# dtoverlay=vc4-kms-v3d
gpu_mem=128
```

**保存并重启**：
```bash
# Ctrl+O 保存，Ctrl+X 退出
sudo reboot
```

重启后再运行 `startx` 命令，应该能正常显示。

---

## 6. RDP 客户端深度配置

使用 `xfreerdp` 构建启动脚本 (`start_rdp.sh`)。

### 6.1 基础依赖安装

```bash
sudo apt update
sudo apt install --no-install-recommends xserver-xorg xinit xterm freerdp2-x11 pulseaudio fonts-noto-cjk
```

### 6.2 启动脚本（start_rdp.sh）

**完整脚本内容**：

```bash
#!/bin/bash

echo "=== Prepare Connect Windows 11 ==="
echo -n "Please enter windows username: "
read USERNAME
echo -n "Please enter password: "
read -s PASSWORD
echo ""

sudo startx /usr/bin/xfreerdp \
  /v:<Windows主机IP> \
  /u:"$USERNAME" \
  /p:"$PASSWORD" \
  /network:wan \
  /cert:ignore \
  /f \
  /bpp:16 \
  /gdi:hw \
  +auto-reconnect \
  /auto-reconnect-max-retries:20 \
  -menu-anims \
  -wallpaper \
  -themes \
  -aero \
  -window-drag \
  +fonts \
  +clipboard \
  +glyph-cache \
  +bitmap-cache \
  +offscreen-cache \
  +async-input \
  +async-update \
  /multitransport \
  /audio-mode:2 \
  /compression-level:2 \
  -- :0 vt7
```

**使用方法**：
```bash
# 赋予执行权限
chmod +x start_rdp.sh

# 运行脚本（会提示输入用户名和密码）
./start_rdp.sh
```

### 6.3 关键参数解析

| 参数 | 作用 | 说明 |
|:-----|:-----|:-----|
| `/v:<Windows主机IP>` | 目标主机地址 | Windows 11 主机的 IP 地址（如 192.168.x.x） |
| `/network:wan` | 广域网模式 | **关键优化参数**，针对高延迟环境优化，禁用大部分特效，详见下方说明 |
| `/cert:ignore` | 忽略证书验证 | 跳过 SSL 证书验证（内网环境） |
| `/f` | 全屏模式 | **Fullscreen**，让远程桌面铺满整个显示器，这是"瘦客户端"的核心参数 |
| `/bpp:16` | 16位色深 | 高彩色显示，平衡画质与性能（32位太吃带宽） |
| `/gdi:hw` | 硬件 GDI 加速 | 配合 KMS 驱动提升绘图效率 |
| `+auto-reconnect` | 自动重连 | 连接断开后自动重新连接 |
| `/auto-reconnect-max-retries:20` | 重连次数 | 最多尝试重连 20 次 |
| `-menu-anims` | 禁用菜单动画 | 关闭菜单动画效果，减少数据传输 |
| `-wallpaper` | 禁用壁纸 | 禁用桌面壁纸，改用纯色背景 |
| `-themes` | 禁用主题 | 禁用 Windows 主题特效 |
| `-aero` | 禁用 Aero 特效 | 禁用窗口透明和毛玻璃效果 |
| `-window-drag` | 禁用窗口拖拽内容 | 拖动窗口时只显示边框，不显示内容 |
| `+fonts` | 字体平滑 | 启用字体平滑渲染 |
| `+clipboard` | 剪贴板共享 | 支持本地与远程之间复制粘贴 |
| `+glyph-cache` | 字形缓存 | 加速文字渲染 |
| `+bitmap-cache` | 位图缓存 | 减少重复图像传输 |
| `+offscreen-cache` | 离屏缓存 | 缓存不可见区域的图像数据 |
| `+async-input` | 异步输入 | 启用异步输入处理 |
| `+async-update` | 异步更新 | 启用异步屏幕更新 |
| `/multitransport` | 多路传输 | 允许同时使用 TCP 控制流和 UDP 数据流 |
| `/audio-mode:2` | 音频模式 | 音频播放模式（0=禁用，1=本地播放，2=不播放但允许录音） |
| `/compression-level:2` | 压缩级别 | RDP 压缩级别（0=不压缩，1=低，2=高，最大值为2） |
| `-- :0 vt7` | X Server 配置 | 在显示 :0 和虚拟终端 vt7 上运行 |

### 6.4 关键参数深度解析

#### /f 全屏模式详解

**含义**: Fullscreen (全屏模式)

**作用**: 让远程桌面铺满整个显示器，遮挡树莓派原本的黑底或其他窗口

**运维场景**: 这是把树莓派变成"瘦客户端"的**核心参数**。加上它，用户感觉自己就在操作一台 Windows 电脑

**切换快捷键**: 
- 全屏/窗口切换: `Ctrl + Alt + Enter`
- 切回树莓派 TTY: 先按 `Ctrl + Alt + Enter` 退出全屏，再按 `Ctrl + Alt + F2`

#### /network 带宽与特效控制

这个参数**非常关键**。它不仅告诉服务器网速多少，实际上控制了 **Windows 11 发送多少特效过来**。

**参数格式**: `/network:[类型]`

**支持的类型（按带宽从低到高）**：

| 类型值 | 含义 | 视觉效果 | 适合 Pi 3B？ |
|:------|:-----|:---------|:------------|
| `modem` | 56kbps 调制解调器 | 极差。无壁纸，无菜单动画，无字体平滑，单色背景 | ❌ 仅调试时用 |
| `broadband-low` | 宽带 (256k-2Mbps) | 低。"性能优先"模式，关闭大部分特效 | ✅ **推荐** (极其流畅) |
| `broadband` | 宽带 (2M-10Mbps) | 中。保留壁纸，关闭透明玻璃特效和阴影 | ✅ **推荐** (平衡之选) |
| `broadband-high` | 高速宽带 (10Mbps+) | 高。开启大部分特效 | ⚠️ 可尝试 |
| `wan` | 广域网 | 针对高延迟优化的宽带模式 | ⚠️ 远程异地用 |
| `lan` | 局域网 (>100Mbps) | 最高。开启所有特效（菜单淡入淡出、阴影、字体抗锯齿） | ❌ 不推荐 (Pi 3B 扛不住) |
| `auto` | 自动检测 | 自动协商 | ❌ 不建议 (容易误判) |

**Pi 3B 推荐配置**:
- **当前配置**：`/network:wan`（针对高延迟环境优化，配合大量禁用特效参数）
- 局域网内且追求画质：`/network:broadband`（平衡性能与画质）
- 性能优先：`/network:broadband-low`（最流畅，牺牲部分视觉效果）
- 不要用：`/network:lan`（特效太多，Pi 3B 扛不住）

**当前配置的性能优化说明**：
- `/bpp:16`：使用16位色深，相比32位减少50%的图像数据传输量
- `/network:wan`：针对高延迟环境优化，自动调整缓冲策略
- `/compression-level:2`：中等压缩级别，平衡 CPU 负载与带宽占用
- `/audio-mode:2`：禁用音频播放，节省带宽和 CPU 资源（仅保留录音功能）
- `-wallpaper -themes -aero -window-drag`：禁用所有视觉特效，显著降低数据传输量
- `+offscreen-cache`：缓存离屏内容，减少重复传输
- `+async-update`：异步更新机制，提升响应速度

**进一步优化建议**：
- 如需音频：将 `/audio-mode:2` 改为 `/audio-mode:1`，但会增加约 10-20% CPU 负载
- 如网络极佳：可尝试 `/bpp:32` 提升画质，但带宽占用翻倍
- 如带宽不足：当前已使用最高压缩级别 `/compression-level:2`，无法进一步提升

---

## 7. 网络传输协议优化（UDP 激活）

**目标**: 让连接使用 UDP (3389) 而非 TCP，以降低延迟和卡顿。

### 7.1 Windows 端设置

#### 防火墙配置
必须放行 UDP 3389 入站。

```powershell
# 管理员 PowerShell 执行
New-NetFirewallRule -DisplayName "Allow RDP UDP" -Direction Inbound -Protocol UDP -LocalPort 3389 -Action Allow
```

#### 组策略检查
确保 `计算机配置 -> 管理模板 -> Windows 组件 -> 远程桌面服务 -> 连接 -> 选择 RDP 传输协议` 未被禁用。

### 7.2 VPN（ZeroTier）优化

如果 RDP 跑在 VPN 内，MTU 分片会导致 UDP 握手失败回退到 TCP。

#### ZeroTier 安装与 Moon 配置

```bash
curl -s https://install.zerotier.com | sudo bash
sudo zerotier-cli join <NetworkID>
# Moon 需执行两次 ID
sudo zerotier-cli orbit <MoonID> <MoonID>
```

#### MTU 调整（解决 UDP 不通）

```bash
# 将 ZeroTier 网卡 MTU 调小
sudo ip link set dev <zt_interface> mtu 1350
```

### 7.3 验证方法

连接状态下，在树莓派 TTY 中执行：

```bash
ss -tunp | grep 3389
# 看到 udp ESTAB 即为成功
```

---

## 8. 终端环境（Xterm）定制

**结论**: Pi 3B 无法运行 Ghostty（缺 OpenGL 3.3）。Xterm 是最佳选择。

### 8.1 启动脚本（start_xterm.sh）

**完整脚本内容**：

```bash
#!/bin/bash
# 启动第二个图形界面，注意显示号是 :1，终端是 vt8
# -geometry 145x40 是为了让 xterm 占满全屏，不然只有左上角一小块
sudo startx /usr/bin/xterm \
  -geometry 145x40 \
  -bg "#282c34" \
  -fg "#abb2bf" \
  -fa 'Monospace' \
  -fs 14 \
  -- :1 vt8
```

**使用方法**：
```bash
# 赋予执行权限
chmod +x start_xterm.sh

# 运行脚本
./start_xterm.sh
```

### 8.2 参数说明

| 参数 | 作用 | 说明 |
|:-----|:-----|:-----|
| `-geometry 145x40` | 窗口尺寸 | 145列x40行，占满 1080p 屏幕 |
| `-bg "#282c34"` | 背景色 | 深灰色背景（类似 VS Code 主题） |
| `-fg "#abb2bf"` | 前景色 | 浅灰色文字 |
| `-fa 'Monospace'` | 字体 | 等宽字体 |
| `-fs 14` | 字体大小 | 14pt 字体 |
| `-- :1 vt8` | X Server 配置 | 在显示 :1 和虚拟终端 vt8 上运行 |

**中文支持**：
如需显示中文，需添加 `-fd` 参数指定中文字体：
```bash
-fd 'Noto Sans Mono CJK SC'
```

**多终端说明**：
- RDP 运行在 `:0 vt7`
- Xterm 运行在 `:1 vt8`
- 两者独立运行，可通过 `Ctrl+Alt+F7` 和 `Ctrl+Alt+F8` 切换

---

## 9. 常见故障排查（Troubleshooting）

### Q: 报错 `0xC000006D` / Login Failure?
**A**: 
- 密码有特殊字符需加单引号 `'`
- 检查是否使用了 Microsoft 账号密码（而非 PIN 码）

### Q: 按 `Ctrl+Alt+F2` 无法切换回树莓派 TTY?
**A**: 
- 键盘被 Windows 捕获了
- 先按 **`Ctrl` + `Alt` + `Enter`** 释放焦点，再切 TTY

### Q: 画面卡顿，拖动窗口残影?
**A**: 
- Pi 3B CPU 瓶颈，当前已使用 `/bpp:16` 和 `/network:wan` 优化
- 如仍卡顿，尝试 `/network:broadband-low`（更激进的性能模式）
- 检查 `gpu_mem` 是否设置为 `128`（显存不足也会卡）
- 当前已使用最高压缩级别 `/compression-level:2`，无法通过提高压缩优化

### Q: 直连也是 TCP?
**A**: 
- 加上 `/sec:nla` 强制握手
- 检查 Windows 注册表 `fClientDisableUDP`

---

## 10. 参考资料（References）

1. **FreeRDP User Manual & Command Line Interface**
   - https://github.com/FreeRDP/FreeRDP/wiki/CommandLineInterface

2. **Raspberry Pi Documentation - Legacy Video Configuration (config.txt)**
   - https://www.raspberrypi.com/documentation/computers/config_txt.html#video-options

3. **ZeroTier Knowledge Base - Router Configuration & MTU**
   - https://docs.zerotier.com/router/

4. **Xterm Control Sequences and Options**
   - https://invisible-island.net/xterm/manpage/xterm.html

---

**文档更新日期**: 2025-12-19
**维护者**: 运维工程师
