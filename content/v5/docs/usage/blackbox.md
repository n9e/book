---
title: "远程探测"
linkTitle: "远程探测"
date: 2021-05-19
description: >
  远程探测这块，包括ICMP、TCP、HTTP，主要用于网络连通性测试，这块blackbox_exporter已经做的挺好了，我们可以直接复用，把数据抓到时序库，夜莺直接去消费即可。
---

TODO: 讲解blackbox的使用，梳理内置大盘和告警策略

# 安装blackbox_exporter
> 01 下载
```shell script
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.19.0/blackbox_exporter-0.19.0.linux-amd64.tar.gz
```

> 02 使用默认的配置启动 blackbox.yml
- 当然也可以基于 tcp、http、icmp三种基础探针自己封装探测模块
- 默认配置如下

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

> 03 启动服务
- 下面给出的是systemd配置，路径自己更改
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

> 04 检查blackbox_exporter服务
- 本地或者浏览器 探测下baidu.com试试
```shell script
curl 'http://localhost:9115/probe?target=baidu.com&module=http_2xx'
```
- 如果能出现metrics结果说明服务正常
- 至此blackbox的配置安装结束

# 配置prometheus 添加相关采集job

> 01 配置prometheus配置文件，按需添加探测job

- 注意下面的配置都需要配置 prometheus 的relabel能力，具体原理请自行查阅
- http接口的探测配置，使用http_2xx模块，GET方法
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

- ssh联通性的探测配置 使用 ssh_banner模块，底层是tcp探针

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

- ping监控：使用icmp模块
```yaml

	
  - job_name: 'blackbox_icmp'
    metrics_path: /probe
    params:
      module: [ssh_banner]  # Look for a HTTP 200 response.
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

- tcp端口探测 使用tcp_connect模块
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


> 02 观察prometheus target页面
- targets页面中出现上述配置的探测方法则为成功
- prometheus ui上查询probe开头的metrics，如果查询到则成功


# 导入夜莺内置 blackbox_exporter大盘就能看到图了
# 导入夜莺内置告警策略 blackbox_exporter 就能配置告警了

