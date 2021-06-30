**1.环境介绍**
在两台物理机上搭建一个server + client 的采集和展现的基本环境来熟悉一下prometheus的基本实现和操作

|nodename|ip|
|---|---|
| client| 10.255.72.193|
| server| 10.255.72.206|


**2.在client端安装node_exporter**

``` bash
wget https://github.com/prometheus/node_exporter/releases/download/v0.16.0/node_exporter-0.16.0.linux-amd64.tar.gz
tar zxvf node_exporter-0.16.0.linux-amd64.tar.gz
cd node_exporter-0.16.0.linux-amd64
./node_exporter
```

**3.在server端安装prometheus**

``` bash
wget https://github.com/prometheus/prometheus/releases/download/v2.2.1/prometheus-2.2.1.linux-amd64.tar.gz
tar zxvf prometheus-2.2.1.linux-amd64.tar.gz
cd prometheus-2.2.1.linux-amd64
```

- 添加配置

``` yaml
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'node-10.255.72.193'
    static_configs:
      - targets: ['10.255.72.193:9100']
```
- 启动prometheus

``` bash
mkdir -p /data/prometheus/
./prometheus --config.file=prometheus.yml --storage.tsdb.path=/data/prometheus
```

**4.访问web端**

``` bash
http://10.255.72.206:9090/graph/

```
