# Prometheus + Grafana 服务器监控平台

基于阿里云 ECS 搭建的 3 节点监控系统，采集 CPU、内存、磁盘、网络核心指标。

## 架构 / Architecture /ˈɑːrkɪtektʃər/（阿-ki-泰克-彻）
aliyun1 (控制节点/Control Node)
├── Prometheus /prəˈmiːθiəs/ (端口 9090) ← 时序数据库，抓取存储指标
└── Grafana /ˈɡræfənə/ (端口 3000) ← 可视化仪表盘
aliyun2 / aliyun3 (被管节点/Managed Nodes)
└── Node Exporter /noʊd ɪkˈspɔːrtər/ (端口 9100) ← 暴露系统级指标

## 技术栈 / Tech Stack

| 组件 | 版本 | 作用 |
|:---|:---|:---|
| Prometheus | v2.53.1 | 时序数据库，每 15s 主动抓取（scrape /skreɪp/）指标 |
| Grafana | v10.4.x | 可视化展示，导入官方仪表盘 ID `1860` |
| Node Exporter | v1.8.2 | 系统指标导出器 |
| Ansible | 2.x | 批量部署 Node Exporter 到被管节点 |

## 部署节点 / Nodes

| 节点 | 内网 IP | 角色 |
|:---|:---|:---|
| aliyun1 | 172.25.112.181 | Prometheus + Grafana |
| aliyun2 | 172.24.47.102 | Node Exporter + Nginx |
| aliyun3 | 172.24.x.x | Node Exporter + Nginx |

> 注意：aliyun3 的 IP 请改成真实地址

## 截图 / Screenshots

<img width="1919" height="1020" alt="image" src="https://github.com/user-attachments/assets/a12d8ccc-d04f-480c-95e2-ba721b105159" />


## 踩坑记录 / Troubleshooting /ˈtrʌblʃuːtɪŋ/（川-波-修-汀）

1. **安全组端口未开放**：Grafana 外网无法访问，需在阿里云控制台开放 9090/3000/9100 端口并限制来源 IP

2. **Node Exporter 安装超时**：国内服务器从 GitHub 下载二进制包大概率超时，改用 `apt install prometheus-node-exporter` 解决

3. **Prometheus 配置 targets**：初始只配了 localhost，后补充 aliyun2/aliyun3 私网 IP，重启服务后 3 节点全部 UP

4. **Grafana 仪表盘"无数据"**：时间范围选"过去24小时"导致，改为"过去5分钟"后正常显示

5. **Grafana 官方源下载慢**：300MB 安装包从官方源下载需 7 小时，换清华大学镜像源后 30 秒完成

## 快速开始 / Quick Start

```bash
# 1. 安装 Prometheus（控制节点）
sudo apt install prometheus -y

# 2. 修改配置，添加被管节点
sudo nano /etc/prometheus/prometheus.yml

# 3. 重启加载配置
sudo systemctl restart prometheus

# 4. 安装 Grafana
sudo apt install grafana -y
sudo systemctl start grafana-server
Ansible 批量部署 Node Exporter
yaml
---
- name: Install Node Exporter via apt
  hosts: webservers
  become: yes

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
      changed_when: false

    - name: Install Node Exporter
      apt:
        name: prometheus-node-exporter
        state: present

    - name: Ensure Node Exporter is running
      systemd:
        name: prometheus-node-exporter
        state: started
        enabled: yes
```

Prometheus 配置示例
```yaml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets:
        - '172.25.112.181:9100'
        - '172.24.47.102:9100'
访问地址
Prometheus: http://8.163.26.172:9090
Grafana: http://8.163.26.172:3000
监控项目关联仓库：github.com/bileimusi/ansible-ops

```
