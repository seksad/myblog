---
title: grafana+prometheus简单监控
description: grafana+prometheus简单监控 
date: 2024-12-17 23:42:40+0000
categories:
  - linux
tags:
  - grafana
  - docker
---

# 安装prometheus

创建`/opt/prometheus/config`和`/opt/prometheus/data`文件夹。

```bash
mkdir /opt/prometheus/config
mkdir /opt/prometheus/data
```

将下面2个文件，`prometheus.yml`和`alert.rules.yml`放到`.config`文件夹下面。

这是`prometheus.yml`内容

```prometheus.yml
global:
  scrape_interval: 15s

rule_files:
  - /etc/prometheus/alert.rules.yml

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

alerting:
  alertmanagers:
  - static_configs:
    - targets: ['alertmanager:9093']
```

这是`alert.rules.yml`内容

```alert.rules.yml
groups:
  - name: example
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: page
        annotations:
          summary: "Instance {{$labels.instance}} down"
          description: "{{$labels.instance}} of job {{$labels.job}} has been down for more than 5 minutes."
```

然后，运行prometheus

```bash
docker run  -d --name prometheus \
--restart=unless-stopped \
--privileged=true \
-u root \
-p 9090:9090 \
-v /opt/prometheus//config:/etc/prometheus \
-v /opt/prometheus//data:/prometheus \
prom/prometheus:latest
```

此时，访问`ip:9090`可以看到prometheus的网页

## 安装node-exporter(收集主机数据)

```bash
docker run -d \
--name node-exporter \
--restart=unless-stopped \
-p 9100:9100 \
-v "/proc:/host/proc:ro" \
-v "/sys:/host/sys:ro" \
-v "/:/rootfs:ro" \
prom/node-exporter:latest
```

访问`http://ip:9100/metrics`，即可看到监控的主机数据

将`node-exporter`与`prometheus`添加到同一网络，然后修改`prometheus.yml`,改为如下

```prometheus.yml
global:
  scrape_interval: 15s

rule_files:
  - /etc/prometheus/alert.rules.yml

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: linux
    static_configs:
      - targets: ['node-exporter:9100']
        labels:
          instance: localhost
   #  - targets: ['192.168.1.22:9100']  
   #这里添加targets，可以使用Prometheus监控其他装有node_exporter的节点，单节点则不需要
   #    labels:
   #      instance: 192.168.1.22

alerting:
  alertmanagers:
  - static_configs:
    - targets: ['alertmanager:9093']
```

## 安装 mysqld-exporter(收集主机数据)

```bash
docker run -d \
--name mysqld_exporter \
--restart=unless-stopped \
-p 9104:9104 \
-e DATA_SOURCE_NAME="root:Password123@(172.17.0.2:3306)/" \
prom/mysqld-exporter:latest
```

访问`http://ip:9104/metrics`，即可看到监控的主机数据

将`mysqld-exporter`与`prometheus`添加到同一网络，然后修改`prometheus.yml`,改为如下

```prometheus.yml
global:
  scrape_interval: 15s

rule_files:
  - /etc/prometheus/alert.rules.yml

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: linux
    static_configs:
      - targets: ['node-exporter:9100']
        labels:
          instance: localhost
   #  - targets: ['192.168.1.22:9100']  
   #这里添加targets，可以使用Prometheus监控其他装有node_exporter的节点，单节点则不需要
   #    labels:
   #      instance: 192.168.1.22
  - job_name: mysqld
    static_configs:
      - targets: ['172.17.0.3:9104']
        labels:
          instance: mysql-exporter
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['alertmanager:9093']
```

## 安装cadvisor(收集容器数据)

```bash
docker run \
-v /:/rootfs:ro \
-v /var/run:/var/run:ro \
-v /sys:/sys:ro \
-v /var/lib/docker/:/var/lib/docker:ro \
-v /dev/disk/:/dev/disk:ro \
-p 8080:8080 \
-d --name=cadvisor \
--privileged \
--restart=unless-stopped \
google/cadvisor:latest
```

访问`http://ip:8080/metrics`，即可看到监控的容器的数据

将`cadvisor`与`prometheus`添加到同一网络，然后修改`prometheus.yml`,改为如下

```prometheus.yml
global:
  scrape_interval: 15s

rule_files:
  - /etc/prometheus/alert.rules.yml

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: linux
    static_configs:
      - targets: ['node-exporter:9100']
        labels:
          instance: localhost
   #  - targets: ['192.168.1.22:9100']  
   #这里添加targets，可以使用Prometheus监控其他装有node_exporter的节点，单节点则不需要
   #    labels:
   #      instance: 192.168.1.22
  - job_name: mysqld
    static_configs:
      - targets: ['172.17.0.3:9104']
        labels:
          instance: mysql-exporter
  - job_name: cadvisor
    static_configs:
      - targets: ['cadvisor:8080']
        labels:
          instance: cAdvisor

alerting:
  alertmanagers:
  - static_configs:
    - targets: ['alertmanager:9093']
```

# 安装grafana

创建目录`/opt/grafana/`

```bash
mkdir /opt/grafana
```

然后启动grafana

```bash
docker run -d \
-p 3000:3000 \
--name grafana \
-v /opt/grafana/data:/var/lib/grafana \
-v /etc/localtime:/etc/localtime \
grafana/grafana:latest
```

然后访问`http://ip:3000`进入grafana，

pass

### 添加数据源->选择prometheus

### 导入面板

输入id

12633

8919

更多grafana模板: https://grafana.com/grafana/dashboards 搜索 相应 dashboards的id如**8919，12227**
