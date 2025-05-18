---

title: Grafana + Prometheus 实现 PVE Host 和 Guest 的监控
created: 2025-05-18
tags:
    - Grafana
    - Prometheus
    - PVE
    - Monitor

---

使用 `prometheus-pve-exporter` 通过调用 `PVE API` 获取 `PVE` 的运行指标。

使用 `Prometheus` 存储 `PVE` 的运行指标。

使用 `Grafana` 作为监控数据的展示。

## 参考

- [Prometheus PVE Exporter](https://github.com/prometheus-pve/prometheus-pve-exporter)
- [Proxmox Prometheus Exporter Questions #8](https://forum.proxmox.com/threads/proxmox-prometheus-exporter-questions.74862/#post-334340)
- [proxmox-via-prometheus-dashboard](https://github.com/mittelab/proxmox-via-prometheus-dashboard)
- [Pve使用外部监控Grafana+Prometheus](https://foxi.buduanwang.vip/virtualization/pve/615.html/)

## 安装 `prometheus-pve-exporter`

[prometheus-pve-exporter](https://github.com/prometheus-pve/prometheus-pve-exporter)

在任意可访问到 `PVE` 的机器上，执行以下命令安装 `prometheus-pve-exporter`。

Docker Compose:

```yaml
services:
  prometheus-pve-exporter:
    image: prompve/prometheus-pve-exporter
    container_name: prometheus-pve-exporter
    init: true
    ports:
      - "9221:9221"
    volumes:
      - ./pve.yml:/etc/prometheus/pve.yml
    restart: unless-stopped
```

## 配置 `prometheus-pve-exporter`

在 `pve.yml` 文件中，配置 `PVE` 的访问信息。

```yaml
default:
    user: your-username@pve
    token_name: "your-token-name"
    token_value: "your-token-value"
    verify_ssl: false
```

建议在 `PVE` 中创建一个 `API` 用户，并使用该用户创建一个 `API` 令牌。
创建 `API` 用户和 `API` 令牌的步骤如下：

1.使用 `Admin` 用户登录 `PVE` 的 `Web UI`。
2.在 `Datacenter` 页面，点击 `Users` 菜单。
3.点击 `Add` 按钮，添加一个 `PVE` 用户，`Realm` 选择 `Proxmox VE authentication server`，`Username` 输入用户名，`Password` 输入密码，点击 `Add` 按钮。
将用户名保存至 `pve.yml` 文件 `user` 中。

![[Pasted image 20250518104314.png]]

4.在 `API Tokens` 页面，点击 `Add` 按钮，添加一个 `API` 令牌，`User` 选择刚刚创建的用户，`Token Name` 输入令牌名称，`Privilege Separation` 保持默认的 `enable`，点击 `Add` 按钮。
添加成功后，会自动生成一个 `API Token` 值，复制该值并保存至 `pve.yml` 文件 `token_value` 中。同时将 `Token Name` 保存至 `pve.yml` 文件 `token_name` 中。

![[Pasted image 20250518104505.png]]

5.在 `Permissions` 页面，点击 `Add` 按钮，选择 `User Permission`，`Path` 选择 `/`，`User` 选择刚刚创建的用户，`Role` 选择 `PVEAuditor`，`Propagate` 保持默认的 `enable` 状态。

![[Pasted image 20250518104824.png]]

6.点击 `Add` -> `API Token Permission`，为 `API Token` 添加同样的权限。

![[Pasted image 20250518105118.png]]

至此，`PVE` 端的用户权限和 `API Token` 权限配置完成。`prometheus-pve-exporter` 的配置完成。

若可以正常访问 `http://<pve-exporter-ip>:9221/pve?cluster=1&node=1&target=<pve-host-ip>`，并返回所有指标，则说明 `prometheus-pve-exporter` 配置成功。

## 安装 `Prometheus` 和 `Grafana`

[[基于 Docker Compose 的 Prometheus 和 Grafana 的安装和配置]]

## 配置 `Prometheus`

在 `prometheus.yml` 文件中，添加以下内容：

如果 `PVE Host` 和 `PVE Exporter` 不在同一台机器上，添加如下的配置，指定 `replacement` 为 `PVE Exporter` 的 `IP` 和 `Port`。

```yaml
scrape_configs:
  - job_name: 'pve'
    static_configs:
      - targets:
        - 192.168.0.20  # Proxmox VE host node.
    metrics_path: /pve
    params:
      module: [default]
      cluster: ['1']
      node: ['1']
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.0.31:9221  # PVE exporter.
```

如果 `PVE Exporter` 运行在 `PVE Node` 上，则添加如下的配置：

```yaml
scrape_configs:
  - job_name: 'pve'
    static_configs:
      - targets:
        - 192.168.1.2:9221  # Proxmox VE node with PVE exporter.
        - 192.168.1.3:9221  # Proxmox VE node with PVE exporter.
    metrics_path: /pve
    params:
      module: [default]
      cluster: ['1']
      node: ['1']
```

## 配置 `Grafana`

确保 `Grafana` 以配置 `Prometheus` 数据源，并在 `WebUI` 上导入如下的 `Dashboard`。

```text
10347-proxmox-via-prometheus
```
