---
title: systemd-analyze - 查看启动所使用的时间
created: 2025-01-05
language: Bash
tags:
    - Linux
---

```bash
systemd-analyze
```

```bash
root@pve-01:/mnt/nas# systemd-analyze
Startup finished in 36.804s (firmware) + 5.846s (loader) + 2.443s (kernel) + 44.113s (userspace) = 1min 29.208s
graphical.target reached after 44.098s in userspace.
```

```bash
systemd-analyze blame
```

![[Pasted image 20250105194639.png]]