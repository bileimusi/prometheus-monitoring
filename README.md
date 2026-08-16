# Prometheus + Grafana 服务器监控平台

基于阿里云 ECS 搭建的节点级监控系统，采集 CPU、内存、磁盘、网络核心指标。

## 架构 / Architecture

aliyun1 (控制节点/Control Node)
├── Prometheus (端口 9090) ← 时序数据库，抓取存储指标
└── Grafana (端口 3000) ← 可视化仪表盘

aliyun2 / aliyun3 (被管节点/Managed Nodes)
└── Node Exporter (端口 9100) ← 暴露系统级指标

## 技术栈 / Tech Stack

| 组件 | 版本 | 作用 |
|:---|:---|:---|
| Prometheus | v2.53.1 | 时序数据库，每 15s 主动抓取（scrape）指标 |
| Grafana | v10.4.x | 可视化展示，导入官方仪表盘 ID `1860` |
| Node Exporter | v1.8.2 | 系统指标导出器 |
| Ansible | 2.x | 批量部署 Node Exporter 到被管节点 |

## 部署节点 / Nodes

| 节点 | 内网 IP | 角色 |
|:---|:---|:---|
| aliyun1 | 172.24.47.99 | Prometheus + Grafana |
| aliyun2 | 172.25.112.181 | Node Exporter + Nginx |
| aliyun3 | 172.24.47.102 | Node Exporter + Nginx |

## 踩坑记录 / Troubleshooting

1. **安全组端口未开放**：Grafana 外网无法访问，需在阿里云控制台开放 9090/3000/9100 端口并限制来源 IP

 2.Node Exporter 安装超时：国内服务器从 GitHub 下载二进制包大概率超时，改用 apt install prometheus-
 node-exporter 解决

3. **Prometheus 配置 targets**：初始只配了 localhost，后补充 aliyun2/aliyun3 私网 IP

4. **Grafana 仪表盘"无数据"**：时间范围选"过去24小时"导致，改为"过去5分钟"后正常显示

## 截图 / Screenshots

<img width="1882" height="968" alt="image" src="https://github.com/user-attachments/assets/b0f9ccce-8f5c-4fec-ab04-406aa77c4d6f" />


## 访问地址

- **Prometheus**: http://8.163.26.172:9090
- **Grafana**: http://8.163.26.172:3000

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
