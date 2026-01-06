---
title: Rime 万象更新工具
created: 2026-01-06
tags:
  - Rime
  - Automation
  - MacOS
---

## 简介

本文档介绍如何使用 Rime 万象更新工具实现 Rime 输入法的自动更新，并通过 macOS 的 LaunchAgents 实现定时自动更新。

## 步骤

### 1. 下载更新脚本

从 GitHub 仓库下载用于更新的 Python 脚本：

- 仓库地址：https://github.com/rimeinn/rime-wanxiang-update-tools
- 脚本文件：`rime-wanxiang-update-win-mac-ios-android.py`

### 2. 放置脚本

将下载的脚本放置到任意目录下，例如：

```bash
<YOUR_SCRIPTS_DIR>/rime-wanxiang-update-win-mac-ios-android.py
```

### 3. 创建虚拟环境并安装依赖

#### 3.1 创建 requirements.txt

在脚本所在目录创建 `requirements.txt` 文件，内容如下：

```txt
# RIME万象输入法更新脚本依赖
# 适用于 macOS/Linux/Windows

# HTTP请求库
requests>=2.31.0

# 进度条显示
tqdm>=4.66.0
```

#### 3.2 创建虚拟环境并安装依赖

```bash
# 创建虚拟环境
python3 -m venv rime_env

# 激活虚拟环境
source rime_env/bin/activate

# 安装依赖
pip install -r requirements.txt
```

### 4. 首次运行并配置

#### 4.1 确认虚拟环境

运行脚本前，确保已激活虚拟环境：

```bash
source rime_env/bin/activate
```

#### 4.2 首次运行脚本

```bash
python rime-wanxiang-update-win-mac-ios-android.py
```

首次运行时，脚本会进入交互界面：
- 选择前端引擎
- 选择相关配置选项

#### 4.3 修改配置文件

运行完成后，会在当前目录下生成 `settings.ini` 文件。需要修改以下关键配置：

```ini
# 不使用镜像源，直接从 GitHub 下载更新内容
use_mirror = false

# 启用自动更新模式，下次运行时直接执行更新操作
# 设置为 true 后，后续运行将跳过交互界面
auto_update = true

# 添加 GitHub Token 以避免 403 错误
# 可以提高 GitHub API 的访问次数和频率
github_token = <YOUR_GITHUB_TOKEN>
```

**说明**：
- `auto_update = true`：启用后，后续执行将直接根据配置文件执行更新，不再进入交互界面，这对于定时自动更新非常重要
- `github_token`：可选但推荐配置，用于提高 API 访问限制

### 5. 配置 LaunchAgents 定时任务

#### 5.1 创建 plist 文件

在 `~/Library/LaunchAgents/` 目录下创建 `com.rime.autoupdate.plist` 文件：

**文件路径**：`~/Library/LaunchAgents/com.rime.autoupdate.plist`

**文件内容**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.rime.autoupdate</string>

    <key>ProgramArguments</key>
    <array>
        <!-- 使用虚拟环境中的Python -->
        <string><YOUR_VENV_PATH>/bin/python</string>
        <string><YOUR_SCRIPT_PATH>/rime-wanxiang-update-win-mac-ios-android.py</string>
    </array>

    <key>WorkingDirectory</key>
    <string><YOUR_SCRIPT_DIR></string>

    <key>StartCalendarInterval</key>
    <dict>
        <key>Weekday</key>
        <integer>0</integer>
        <key>Hour</key>
        <integer>3</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>

    <key>StandardOutPath</key>
    <string><YOUR_LOG_DIR>/rime_update.log</string>

    <key>StandardErrorPath</key>
    <string><YOUR_LOG_DIR>/rime_update_error.log</string>
</dict>
</plist>
```

**配置说明**：
- `<YOUR_VENV_PATH>`：虚拟环境路径，例如 `/Users/username/scripts/rime_env`
- `<YOUR_SCRIPT_PATH>`：脚本完整路径
- `<YOUR_SCRIPT_DIR>`：脚本所在目录
- `<YOUR_LOG_DIR>`：日志输出目录
- `StartCalendarInterval`：定时配置，示例为每周日凌晨 3:00 执行

#### 5.2 加载并测试服务

```bash
# 检查服务是否已加载
launchctl list | grep com.rime.autoupdate

# 加载服务
launchctl load ~/Library/LaunchAgents/com.rime.autoupdate.plist

# 再次检查服务状态
launchctl list | grep com.rime.autoupdate

# 查看服务详细信息
launchctl list com.rime.autoupdate

# 手动启动服务进行测试
launchctl start com.rime.autoupdate
```

#### 5.3 检查日志输出

执行完服务后，检查日志文件：

```bash
# 查看标准输出日志
cat <YOUR_LOG_DIR>/rime_update.log

# 查看错误日志
cat <YOUR_LOG_DIR>/rime_update_error.log
```

确认日志文件已正常创建并有输出内容。

## 注意事项

1. **虚拟环境路径**：LaunchAgents 配置中必须使用虚拟环境中的 Python 解释器完整路径
2. **工作目录**：`WorkingDirectory` 必须设置为脚本所在目录，确保 `settings.ini` 能被正确读取
3. **日志目录**：需要提前创建日志输出目录，否则可能导致任务执行失败
4. **权限问题**：确保 plist 文件和脚本都有执行权限
5. **GitHub Token**：建议配置 GitHub Personal Access Token，避免 API 限流

## 卸载服务

如需停止并卸载服务：

```bash
# 停止服务
launchctl stop com.rime.autoupdate

# 卸载服务
launchctl unload ~/Library/LaunchAgents/com.rime.autoupdate.plist

# 删除 plist 文件
rm ~/Library/LaunchAgents/com.rime.autoupdate.plist
```