---

title: 更新 MacOS 下的 Rime 的 librime
created: 2026-01-04
tags:
  - MacOS
  - Rime
  - Squirrel
  - InputMethod
  - Maintenance

---

## 概述

本文档说明如何在 macOS 下更新 Squirrel（鼠须管）输入法的 librime 库文件。

## Squirrel Frameworks 目录位置

Squirrel 的动态库文件位于以下目录：

```
/Library/Input Methods/Squirrel.app/Contents/Frameworks/
```

## 需要替换的文件

更新时需要替换以下 dylib 文件：

1. `/Library/Input Methods/Squirrel.app/Contents/Frameworks/librime.1.dylib`
2. `/Library/Input Methods/Squirrel.app/Contents/Frameworks/rime-plugins/librime-lua.dylib`
3. `/Library/Input Methods/Squirrel.app/Contents/Frameworks/rime-plugins/librime-octagram.dylib`
4. `/Library/Input Methods/Squirrel.app/Contents/Frameworks/rime-plugins/librime-predict.dylib`

## 获取最新版本

### 1. 访问 GitHub Release 页面

访问 librime 官方仓库的 Release 页面：
```
https://github.com/rime/librime/releases
```

### 2. 下载 macOS 版本

找到最新的 Release，下载 macOS universal 版本的压缩包，例如：
```
rime-75bc43a-macOS-universal.tar.bz2
```

### 3. 解压文件

```bash
tar -xjf rime-75bc43a-macOS-universal.tar.bz2
```

## 替换文件步骤

### 1. 备份原文件（推荐）

```bash
sudo cp -r "/Library/Input Methods/Squirrel.app/Contents/Frameworks" "/Library/Input Methods/Squirrel.app/Contents/Frameworks.backup"
```

### 2. 替换 dylib 文件

将解压后得到的四个文件分别替换到对应目录。

#### 方法一：使用命令行

**文件 1：librime.1.dylib**
```bash
sudo cp librime.1.dylib "/Library/Input Methods/Squirrel.app/Contents/Frameworks/librime.1.dylib"
```

**文件 2：librime-lua.dylib**
```bash
sudo cp librime-lua.dylib "/Library/Input Methods/Squirrel.app/Contents/Frameworks/rime-plugins/librime-lua.dylib"
```

**文件 3：librime-octagram.dylib**
```bash
sudo cp librime-octagram.dylib "/Library/Input Methods/Squirrel.app/Contents/Frameworks/rime-plugins/librime-octagram.dylib"
```

**文件 4：librime-predict.dylib**
```bash
sudo cp librime-predict.dylib "/Library/Input Methods/Squirrel.app/Contents/Frameworks/rime-plugins/librime-predict.dylib"
```

#### 方法二：使用 Finder（需要提权）

1. 按 `Cmd + Shift + G` 打开"前往文件夹"对话框
2. 输入路径：`/Library/Input Methods/Squirrel.app/Contents/Frameworks/`
3. 找到目标文件位置
4. 将新文件拖拽到对应位置进行替换：
   - `librime.1.dylib` → `/Library/Input Methods/Squirrel.app/Contents/Frameworks/`
   - `librime-lua.dylib` → `/Library/Input Methods/Squirrel.app/Contents/Frameworks/rime-plugins/`
   - `librime-octagram.dylib` → `/Library/Input Methods/Squirrel.app/Contents/Frameworks/rime-plugins/`
   - `librime-predict.dylib` → `/Library/Input Methods/Squirrel.app/Contents/Frameworks/rime-plugins/`
5. 系统会提示需要管理员权限，点击"鉴定"并输入密码

## 重启 Squirrel 使更新生效

### 方法一：通过活动监视器（Activity Monitor）

1. 打开"活动监视器"（Activity Monitor）
2. 搜索 "Squirrel" 进程
3. 选中该进程，点击左上角的"X"按钮强制退出
4. 使用系统输入法切换快捷键（通常是 `Control + Space` 或 `Cmd + Space`）切换到 Squirrel
5. Squirrel 会自动重新启动，新的 librime 库文件即生效

### 方法二：通过命令行

```bash
# 杀掉 Squirrel 进程
killall Squirrel

# 然后通过输入法切换重新启动 Squirrel
```

## 注意事项

- 替换系统文件需要管理员权限（`sudo`）
- 建议在替换前备份原文件
- 确保下载的版本与你的 macOS 系统兼容（通常选择 universal 版本）
- 如果更新后出现问题，可以从备份恢复原文件