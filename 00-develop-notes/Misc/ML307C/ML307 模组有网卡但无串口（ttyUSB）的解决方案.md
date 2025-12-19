---
title: ML307 模组有网卡但无串口（ttyUSB）的解决方案
created: 2025-12-19
tags:
    - ML307
    - LTE
    - Linux驱动
    - USB串口
    - 硬件调试
    - 系统配置
---

## ❌ 问题现象
在 Linux 环境下接入 4G/5G 模组（如 ML307、OneMO、移远等），出现以下典型“半驱动”状态：
1.  **网卡正常**：使用 `ifconfig` 或 `ip addr` 可以看到 `eth1` 或 `usb0` 网卡（RNDIS/ECM驱动已加载），网络通讯可能正常。
2.  **串口缺失**：执行 `ls /dev/ttyUSB*` 没有任何输出。
3.  **影响**：无法通过串口发送 AT 指令（如查询信号、切换卡槽、短信操作等）。

## 🔍 问题分析
**原因**：内核作为通用驱动识别了设备的 USB 网络接口（RNDIS/ECM），但内核的 USB 串口驱动（`option` 或 `usb_wwan`）的 ID 列表中尚未包含该模组的 **VID:PID**，导致未能自动挂载串口驱动。

**解决思路**：无需重新编译内核或重启系统，通过 **sysfs** 机制进行 **“动态注入”**，强行告知内核将该 VID:PID 识别为串口设备。

---

## 🛠️ 解决方案：动态注入 USB 串口驱动

此方案为运维标准操作，无需重启，即时生效。

### 步骤 1：获取设备 VID 和 PID
通过 `lsusb` 确认模组的物理 ID。

```bash
lsusb
```

> **示例输出**：
> `Bus 001 Device 004: ID 2949:8247 ...`
>
> *在此例中：VID = **2949**, PID = **8247***

### 步骤 2：加载基础驱动模块
确保内核已加载 `option` 和 `usb_wwan` 模块。如果未加载，后续的注入路径将不存在。

```bash
sudo modprobe usb_wwan
sudo modprobe option
```

### 步骤 3：写入 ID 到驱动注册表（核心步骤）
将步骤 1 中获取的 VID 和 PID 写入到 `option` 驱动的 `new_id` 文件中。
*(⚠️注意：请将下方命令中的 `2949 8247` 替换为你实际查到的 ID)*

```bash
# 语法：echo "VID PID" > /sys/bus/usb-serial/drivers/option1/new_id
sudo sh -c 'echo "2949 8247" > /sys/bus/usb-serial/drivers/option1/new_id'
```

### 步骤 4：验证与测试
查看设备列表，确认是否生成了 ttyUSB 设备节点。

```bash
ls /dev/ttyUSB*
```

*   **成功标志**：终端立即输出 `/dev/ttyUSB0`、`/dev/ttyUSB1` 等设备文件。
*   **后续操作**：通常 `ttyUSB0` 或 `ttyUSB2` 为 AT 指令口，即可使用 `minicom` 或 `echo` 进行测试。

---

## 💡 运维小贴士

### 1. 关于持久化
此方法是临时的（Runtime），重启服务器后失效。如果需要永久生效，有以下两种方案：

#### 方案 A：使用 rc.local（简单直接）
编辑 `/etc/rc.local` 文件，在 `exit 0` 之前添加：
```bash
# 自动加载 ML307 USB 串口驱动
/sbin/modprobe usb_wwan
/sbin/modprobe option
echo "2949 8247" > /sys/bus/usb-serial/drivers/option1/new_id
```

#### 方案 B：使用 udev 规则（推荐）
创建 `/etc/udev/rules.d/99-ml307-usb.rules` 文件：
```bash
# ML307 USB Serial Driver Auto-Load
ACTION=="add", SUBSYSTEM=="usb", ATTRS{idVendor}=="2949", ATTRS{idProduct}=="8247", RUN+="/sbin/modprobe option", RUN+="/bin/sh -c 'echo 2949 8247 > /sys/bus/usb-serial/drivers/option1/new_id'"
```

重新加载 udev 规则：
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

### 2. 权限报错处理
在步骤 3 中，直接使用 `sudo echo "..." > file` 可能会报 *Permission denied*。

**原因**：重定向符号 `>` 是由 shell 执行的而非 sudo，所以 shell 本身没有权限写入系统文件。

**正确做法**：
```bash
# 方法 1：使用 sh -c（推荐）
sudo sh -c 'echo "2949 8247" > /sys/bus/usb-serial/drivers/option1/new_id'

# 方法 2：使用 tee 命令
echo "2949 8247" | sudo tee /sys/bus/usb-serial/drivers/option1/new_id

# 方法 3：切换到 root 用户
sudo -i
echo "2949 8247" > /sys/bus/usb-serial/drivers/option1/new_id
exit
```

### 3. 多串口设备识别
ML307 模组注册成功后，通常会创建多个 ttyUSB 设备：
- **ttyUSB0**：诊断端口（Diag）
- **ttyUSB1**：GPS NMEA 端口
- **ttyUSB2**：AT 指令端口（**ML307C 主要使用此端口**）⭐
- **ttyUSB3**：PPP 拨号端口
- **ttyUSB4**：调试端口（部分型号）

> **⚠️ 重要提示**：对于 **ML307C** 型号，AT 指令端口通常是 **ttyUSB2**，而非 ttyUSB0！

**如何确定 AT 指令端口**：
```bash
# 对于 ML307C，优先测试 ttyUSB2
echo -e "AT\r" > /dev/ttyUSB2
cat /dev/ttyUSB2

# 如果不确定，逐个测试
for port in /dev/ttyUSB*; do
    echo "Testing $port..."
    echo -e "AT\r" > $port 2>/dev/null && timeout 1 cat $port 2>/dev/null
    echo ""
done

# 或使用 minicom（推荐用于 ML307C）
minicom -D /dev/ttyUSB2
```
正确的端口会响应 `OK`。

### 4. 常见问题排查

#### Q: 执行命令后没有生成 ttyUSB 设备？
**A**: 
- 检查驱动模块是否成功加载：`lsmod | grep option`
- 查看内核日志：`dmesg | tail -20`
- 确认 VID:PID 是否正确：`lsusb`
- 尝试重新插拔 USB 设备

#### Q: 提示 "No such file or directory"？
**A**: 
- 确认 option 驱动已加载：`sudo modprobe option`
- 检查路径是否正确：`ls /sys/bus/usb-serial/drivers/`
- 部分系统可能使用 `usb_wwan` 驱动，尝试：
  ```bash
  echo "2949 8247" | sudo tee /sys/bus/usb-serial/drivers/usb_wwan/new_id
  ```

#### Q: 重启后配置失效？
**A**: 
- 参考上面的"持久化"方案，使用 rc.local 或 udev 规则
- 确认 udev 规则文件权限正确：`ls -l /etc/udev/rules.d/99-ml307-usb.rules`

---

## 📚 相关参考

### 常见 4G/5G 模组 VID:PID 列表
| 厂商 | 模组型号 | VID | PID | 备注 |
|:-----|:---------|:----|:----|:-----|
| 中移物联 | ML307 | 2949 | 8247 | 本文重点 |
| 移远 | EC20 | 2c7c | 0125 | 常见型号 |
| 移远 | EC25 | 2c7c | 0121 | 支持全网通 |
| 移远 | EG25-G | 2c7c | 0125 | 4G 全球版 |
| 华为 | ME909s | 12d1 | 15c1 | 企业级模组 |
| 有方 | N58 | 19d2 | 0579 | 工业级 |

### 有用的命令速查
```bash
# 查看 USB 设备
lsusb
lsusb -t

# 查看串口设备
ls -l /dev/ttyUSB*
dmesg | grep ttyUSB

# 查看已加载的驱动
lsmod | grep -E "option|usb_wwan"

# 卸载驱动（调试用）
sudo rmmod option

# 手动触发 udev
sudo udevadm control --reload-rules
sudo udevadm trigger

# 测试 AT 指令
echo -e "AT\r" > /dev/ttyUSB0 && cat /dev/ttyUSB0
```

### 进一步学习
- Linux USB 驱动架构文档：`Documentation/usb/usb-serial.txt`
- udev 规则编写指南：`man udev`
- 内核模块参数：`modinfo option`

---

## 🎯 总结

这个"半驱动"问题在 Linux 环境下使用 4G/5G 模组时非常常见，核心原因是内核的 USB 串口驱动白名单未包含新模组的 VID:PID。

**解决要点**：
1. ✅ 使用 `lsusb` 确认设备 VID:PID
2. ✅ 加载 `option` 和 `usb_wwan` 驱动模块
3. ✅ 通过 sysfs 动态注入 VID:PID
4. ✅ 验证 ttyUSB 设备生成
5. ✅ 配置持久化（udev 规则或 rc.local）

**优势**：
- 无需重新编译内核
- 无需重启系统
- 即时生效，操作简单
- 适用于所有类似的 USB 串口设备

掌握这个方法后，面对任何新的 USB 串口设备都能快速解决驱动问题！