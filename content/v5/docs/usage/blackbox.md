---
title: "远程探测"
linkTitle: "远程探测"
date: 2021-05-19
description: >
  远程探测这块，包括ICMP、TCP、HTTP，主要用于网络连通性测试，这块blackbox_exporter已经做的挺好了，我们可以直接复用，把数据抓到时序库，夜莺直接去消费即可。
---

先给出监控大盘的最终效果如下，对大盘熟悉的朋友也可以自行修改，内置大盘的配置在[这里](https://github.com/didi/nightingale/blob/master/etc/dashboard/blackbox_exporter)，大家安装完夜莺之后，这个配置会出现在n9e-server二进制同级的etc目录下的dashboard下。

![](https://s3-gz01.didistatic.com/n9e-pub/image/blackbox_dash.png)

### 01. 下载探测器

```shell script
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.19.0/blackbox_exporter-0.19.0.linux-amd64.tar.gz
```

### 02. 探测器配置

准备blackbox_exporter所用的配置，姑且叫做blackbox.yml

```shell script
cat <<"EOF" > blackbox.yml
modules:
  http_2xx:
    prober: http
  http_post_2xx:
    prober: http
    http:
      method: POST
  tcp_connect:
    prober: tcp
  pop3s_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^+OK"
      tls: true
      tls_config:
        insecure_skip_verify: false
  ssh_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^SSH-2.0-"
  irc_banner:
    prober: tcp
    tcp:
      query_response:
      - send: "NICK prober"
      - send: "USER prober prober prober :prober"
      - expect: "PING :([^ ]+)"
        send: "PONG ${1}"
      - expect: "^:[^ ]+ 001"
  icmp:
    prober: icmp
 
EOF
```

### 03. 启动探测器

这是使用systemd托管blackbox_exporter，大家也可以自行使用自己习惯的进程托管工具，比如supervisor、god之类的，也可以直接用nohup扔到后台。注意下面的ExecStart这一行配置，样例给的二进制和配置文件都放在了/opt/app/blackbox_exporter，请自行修改适配自己的环境。

```shell script
cat <<"EOF" >  /etc/systemd/system/blackbox_exporter.service
[Unit]
Description=blackbox_exporter Exporter
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/opt/app/blackbox_exporter/blackbox_exporter --config.file=/opt/app/blackbox_exporter/blackbox.yml
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=blackbox_exporter
[Install]
WantedBy=default.target

EOF

systemctl daemon-reload
systemctl restart blackbox_exporter
```

### 04. 检查探测器

我们用blackbox探测器来探测一下`baidu.com`，看看能否返回metrics结果，如果有正常返回，就表示探测器正常

```shell script
curl 'http://localhost:9115/probe?target=baidu.com&module=http_2xx'
```

### 05. 配置抓取器

抓取器就是指prometheus，原理是：prometheus周期性调用blackbox_exporter的接口，让blackbox_exporter对目标地址做探测，对blackbox_exporter的探测结果做解析，生成时序数据写入时序库。

注意下面的配置都需要配置 prometheus 的 relabel 能力，具体原理请自行查阅。http接口的探测配置，使用http_2xx模块，GET方法：

```yaml
  - job_name: 'blackbox_http'
    metrics_path: /probe
    # 传入的参数，选用那个探测模块
    params:
      module: [http_2xx] 
    static_configs:
      - targets:
        - http://prometheus.io    # Target to probe with http.
        - https://www.baidu.com   # Target to probe with https.
        - http://localhost:3000  # Target to probe with http on port 3000.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.
```

SSH连通性的探测配置使用 ssh_banner 模块，底层是 tcp 探针

```yaml
  - job_name: 'blackbox_ssh'
    metrics_path: /probe
    params:
      module: [ssh_banner]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - 172.20.70.205:22    
        - 172.20.70.215:22  
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.
```

PING监控，使用icmp模块

```yaml
  - job_name: 'blackbox_icmp'
    metrics_path: /probe
    params:
      module: [icmp]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - 172.20.70.205
        - 172.20.70.215
        - baidu.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.		
```

TCP端口探测，使用tcp_connect模块

```yaml
  - job_name: 'blackbox_tcp'
    metrics_path: /probe
    params:
      module: [tcp_connect]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - 10.86.76.13:8090 
        - 10.86.76.13:8085 
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  
```

### 06. 检查抓取结果

- 观察prometheus targets页面，targets页面中出现上述配置的探测方法则为成功
- prometheus ui上查询probe开头的metrics，如果查询到则成功

### 07. 导入夜莺模板

- 导入夜莺内置 blackbox_exporter 大盘就能看到图了
- 导入夜莺内置告警策略 blackbox_exporter 就能告警了

下面的系统内置的告警策略，作为[内置策略](https://github.com/didi/nightingale/blob/master/etc/alert_rule/blackbox_exporter)的方式存在：

![](https://s3-gz01.didistatic.com/n9e-pub/image/blackbox_rule.png)
