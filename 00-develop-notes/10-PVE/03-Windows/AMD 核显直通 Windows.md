---
title: AMD 核显直通 Windows
created: 2024-12-23
tags:
    - PVE
    - Windows
    - Passthrough
    - Tutorial
---

由于需要在宿主机上直接输出画面，且 5500GT 不支持 SR-IOV，所以选择在 PVE 中直通 AMD 核显。

## References

- [AMD核显直通显示输出简单方法完整版无定制OVMF开机BIOS启动画面](https://diyforfun.cn/712.html)
- [AMD核显直通显示输出简单方法完整版无定制OVMF开机BIOS启动画面](https://post.smzdm.com/p/axo95nvd/)
- [PVE环境5600G核显直通并输出HDMI接口](https://post.smzdm.com/p/ao9z9d67/)
- [PVE使用AMD CPU 5600G 核显直通](https://blog.csdn.net/qq_42912965/article/details/126815332)
- [PVE下AMD核显直通教程](https://www.bilibili.com/opus/821201771406819398)
- [GMK极摩客K8，PVE8.1下AMD 8845HS WIN11核显直通，HDMI+DP+TYPE-C+音频输出正常;教程适用于GMK极摩客K6](https://post.smzdm.com/p/aqq52rqx/)
- [【NAS】PVE下AMD核显直通和基本配置](https://www.bilibili.com/opus/822884865987838001)
- [PVE7 AMD 5700G 核显直通 (iGPU Passthrough)](https://www.bilibili.com/video/BV11d4y1G7Nk/?vd_source=f812625f00cdd1b06ca2f4281718b552)
- [[深度探索] virt-manager的逆天gpu性能](https://bbs.deepin.org/zh/post/273171)

## PVE 配置

### 硬件

- CPU: AMD 5500GT
- MB: MSI B550M MORTAR MAX WIFI

### 系统

- 8.3.0 (running kernel: 6.8.12-4-pve)

## 配置步骤

### 1. 修改 BIOS 设置

- 开启 SVM（Secure Virtual Machine），默认开启
- 开启 IOMMU（Auto/Enable 均可），默认 Auto
- 开启 CMS（Compatibility Support Module），开启后启动模式会自动设置为 Legacy+UEFI，并且 Secure Boot，Resizable BAR 都会被禁用

### 2. 修改 GRUB 配置

```bash
vi /etc/default/grub
```

- 添加 `initcall_blacklist=sysfb_init iommu=pt amd_iommu=on pcie_acs_override=downstream,multifunction` 到 `GRUB_CMDLINE_LINUX_DEFAULT` 中，如下所示：
  - `initcall_blacklist=sysfb_init` 防止系统占用核显
  - `iommu=pt` 设置 IOMMU 为直通模式 Pass Through
  - `amd_iommu=on` 开启 AMD IOMMU
  - `pcie_acs_override=downstream,multifunction` 用于分离 PCIe 设备，使其能够被单独直通，否则会导致整个 IOMMU 组被直通

> 注意：1、pve内核经常更新，amd_iommu=on有的内核已经内置，因此不需要上面加amd_iommu=on。如果遇到更新pve内核后，提示iommu没有开启，请手动在以上此处添加amd_iommu=on，并update-grub重启后即可解决。
> 2、initcall_blacklist=sysfb_init 启动时运行黑名单内添加项，在Intel的机型中，此项非必要添加，但是在amd机型中建议添加，否则会影响核显直通后的性能，比如4K60Hz降低到30Hz。
> 3、pcie_acs_override=downstream,multifunction PCI ACS覆盖参数避免死机设置为两者/多用途，避免PVE虚拟Win开关机时死机。

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet initcall_blacklist=sysfb_init iommu=pt amd_iommu=on pcie_acs_override=downstream,multifunction"
# 移除 iommu=pt 和 amd_iommu=on 参数后，核显依旧可以正常直通，采用下方的配置
GRUB_CMDLINE_LINUX_DEFAULT="quiet initcall_blacklist=sysfb_init pcie_acs_override=downstream,multifunction"
```

- 更新 GRUB 配置

```bash
update-grub
```

### 3. 修改模块黑名单

```bash
vi /etc/modprobe.d/pve-blacklist.conf
```

```bash
# This file contains a list of modules which are not supported by Proxmox VE 

# nvidiafb see bugreport https://bugzilla.proxmox.com/show_bug.cgi?id=701
blacklist nvidiafb

# block AMD driver
# blacklist radeon
blacklist amdgpu

# block NVIDIA driver
# blacklist nouveau
# blacklist nvidia
# blacklist nvidiafb

# block INTEL driver
# blacklist snd_hda_intel
# blacklist snd_hda_codec_hdmi
# blacklist i915

options vfio_iommu_type1 allow_unsafe_interrupts=1
```

- blacklist `radeon` 和 `amdgpu` 防止 AMD 核显被系统占用
- options `vfio_iommu_type1 allow_unsafe_interrupts=1` 允许不安全的中断

### 4. 添加内核模块

```bash
vi /etc/modules
```

```bash
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

### 5. 更新initramfs

```bash
update-initramfs -u -k all
```

### 6. 重启 PVE

```bash
reboot
```

**确保 PVE 重启之后可以通过 WebUI 正常访问，若无法正常访问，参考 [[通过 Live CD 恢复 PVE]] 进行修复**

PVE 重启后，如果 WebUI 可以正常访问，输出的画面会停留在如下界面。此为正常现象。

> 注意：在设备黑名单中添加 amdgpu，并 grub 里开启引导自动加载后，PVE 启动时会卡在以下界面。是正常的！等待一会只要 PVE 可以通过网页访问到 WebUI 管理页面就是正常没问题的。
因为PVE开机时会按照设备黑名单设置的，释放对核显的控制权。这样才能在核显直通给虚拟Win的时候避免冲突而导致的无法加载驱动，错误代码43

![[2024072201110423.png]]

### 7. 使用 UBU 提取 AMDGopDriver 和 VBIOS 并制作 ROM

[[Tool Guide+News] “UEFI BIOS Updater” (UBU)](https://winraid.level1techs.com/t/tool-guide-news-uefi-bios-updater-ubu/30357)
[edk2-BaseTools-win32](https://github.com/tianocore/edk2-BaseTools-win32)

#### 获取机器PCI信息

```bash
lspci -nnD | grep VGA
lspci -nnD | grep Audio
```

![[Pasted image 20241223232021.png]]

记录下 VGA Controller 和 Audio Controller 的设备 ID，上图中的分别为 `1002:1638` 和 `1002:1637`。

#### 使用 UBU 提取 AMDGOPDriver.efi 和 VBIOS

把主板 BIOS 解压到 UBU 的根目录下，运行UBU.cmd。
然后选择 CPU 的型号，5500GT 属于 5000 Serial，选择 2。

---

![[Pasted image 20241223232956.png]]

选择 2，Video Onboard

---

![[Pasted image 20241223233410.png]]

选择 X，导出 GOP Driver 和 VBIOS Cezanne 和 VBIOS Renoir
导出的文件在 `./Extracted` 目录下，一共得到 `GOP` 和 `vBIOS` 两个目录

![[Pasted image 20241223233635.png]]

vBIOS 目录有 2 个 dat 文件，分别是 `vbios_1638.dat` 和 `vbios_1636.dat`，根据上述 lspci 获取的设备 ID，选择对应的 dat 文件。这里需要使用的是 `vbios_1638.dat`。

#### 使用 edk2-BaseTools-win32 制作 AMDGOPDriver.rom

GOP 目录的文件是 `AMDGopDriver.efi`，把 `AMDGopDriver.efi` 拷贝到 `edk2-BaseTools-win32-master` 目录下，接下在命令行中打开 `edk2-BaseTools-win32` 目录，运行如下命令。
`XXXX` 为具体自己的核显设备 ID，通过上述 `lspci` 获取 5500GT 核显的设备 ID 为 0x1638。

```cmd
EfiRom.exe -f 0x1002 -i 0xXXXX -e AMDGopDriver.efi
```

执行成功后，在 `AMDGopDriver.efi` 同目录下会生成一个 `AMDGopDriver.rom` 文件。

#### 将 AMDGOPDriver.rom 和 VBIOS 上传至 PVE

将 `vbios_1638.dat` 和 `AMDGopDriver.rom` 拷贝到 PVE 的 `/usr/share/kvm` 目录下。

### 8. 新建 Windows 虚拟机并添加核显设备

Windows 虚拟机的创建和上述大步骤之间没有关联，可以在任何时候创建。[[PVE 中 Win11 的安装]]

#### Hardware

![[Pasted image 20241224004503.png]]

- Processors 使用 Host CPU
- BIOS 使用 OVMF (UEFI)
- Machine 使用 q35
- Display 在完成核显直通前，使用 `Default`，直通之后使用 `none`，不添加显卡设备

#### 添加 PCI 设备

添加PCI设备。选择核显编号，勾选 `主GPU`、`ROM-Bar`（默认启用）、`PCI-Express` 这3个选项。

![[Pasted image 20241224011633.png]]

再添加核显声卡（在核显编号后面）

![[Pasted image 20241224011709.png]]

#### 修改 VM 配置

编辑虚拟机配置文件，做如下调整

```bash
vi /etc/pve/qemu-server/{VM-ID}.conf
```

在 `hostpci0` 后面增加 `romfile=vbios_1638.bat`，`hostpci1` 后面增加 `romfile=AMDGopDriver.rom`。如下图所示。
其中的 `vbios_1638.bat` 和 `AMDGopDriver.rom` 是上一步中制作并上传至 PVE `/usr/share/kvm` 下的 ROM 文件。

![[Pasted image 20241224012024.png]]

### 9. 安装 RadeonResetBugFixService 及 AMDGPU 驱动

#### 安装 RadeonResetBugFixService

[Releases · adolfintel/RadeonResetBugFixService](https://github.com/inga-lovinde/RadeonResetBugFix/releases)

下载最新的 `RadeonResetBugFixService.zip`。

解压后放在 C 盘根目录，CMD 管理员模式下运行，安装服务。

```cmd
RadeonResetBugFixService.exe install
```

> （PVE启动之后，第一次启动虚拟Win11可显示引导画面和进入win的画面，之后的重启则是黑屏。请等待2分钟左右RadeonResetBugFixService服务启动，核显会成功加载并工作。关机时RadeonResetBugFixService服务会自动卸载核显，避免下次重启时核显不工作无法进入系统，它会在下次重启时重复上面先开机再启动核显流程，保证每次重启核显都可以正常，且不需要重启PVE，我们仅需要耐心等它2分钟。）

#### 安装 AMDGPU 驱动

[AMD Ryzen™ 5 5500GT Drivers](https://www.amd.com/en/support/downloads/drivers.html/processors/ryzen/ryzen-5000-series/amd-ryzen-5-5500gt.html)

下载最新的 AMDGPU 驱动，安装。

安装完成后，Windows 设备管理器以及任务管理器中会显示 AMD 核显设备。

AMD GPU 可用后，可以在 PVE 中将 `Display` 设置为 `none`，不再使用默认的 `VGA`。

### 10. 添加 USB 设备和音频输出设备等

添加键鼠等 USB 设备和音频输出设备等，使虚拟机可以正常使用。

根据 PVE 的硬件配置，`Family 17h/19h HD Audio Controller` 为主板的音频输出设备，可以直通给虚拟机。

![[Pasted image 20241224105737.png]]

---

USB 设备从上至下分别为：

- 2.4G 无线鼠标（1ea7:0064）
- 键盘（04b4:4042）
- 蓝牙适配器（0e8d:0616）

![[Pasted image 20241224110103.png]]

![[Pasted image 20241224110136.png]]

### 11. 隐藏虚拟化环境（可选）

某些软件可能拒绝运行在虚拟机中（如 DRM 限制、虚拟化检测工具等），此配置可以隐藏虚拟化环境。

```bash
vi /etc/pve/qemu-server/{VM-ID}.conf
```

添加如下配置

```bash
args: -cpu 'host,-hypervisor'
```

- host: 使用宿主机的 CPU 特性。性能最佳，因为它尽可能减少了 CPU 特性模拟。
- -hypervisor: 隐藏虚拟化环境。hypervisor 标志通常用于指示操作系统（如 Linux 或 Windows）当前运行在虚拟化环境中。禁用此标志可以“隐藏”虚拟化环境，使虚拟机中的操作系统认为它运行在物理机上。

---

```bash
cpu: host,hidden=1
```

- hidden=1: 隐藏虚拟化环境。隐藏虚拟化环境，使虚拟机中的操作系统认为它运行在物理机上。
