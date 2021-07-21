---
title: "JMX监控"
linkTitle: "JMX监控"
date: 2021-05-17
description: >
  JMX用于采集JVM的一些监控指标，对于Java类的一些中间件，一般会用这种方式，Java写的业务进程不推荐用JMX方式，建议直接使用micrometer这种SDK埋点，micrometer会自动采集JVM相关监控指标。
---

TODO: 讲解JMX相关采集
# 使用jmx_exporter

## 安装教程

> 01 下载jmx_exporter.jar
```shell script
wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.16.1/jmx_prometheus_javaagent-0.16.1.jar
```


> 02 准备jmx_exporter 基础配置文件
- 下面的配置可以导出一些机器指标
```shell script

cat <<"EOF" > jmx_exporter_common.yaml
---   
lowercaseOutputLabelNames: true
lowercaseOutputName: true
whitelistObjectNames: ["java.lang:type=OperatingSystem"]
blacklistObjectNames: []
rules:
  - pattern: 'java.lang<type=OperatingSystem><>(committed_virtual_memory|free_physical_memory|free_swap_space|total_physical_memory|total_swap_space)_size:'
    name: os_$1_bytes
    type: GAUGE
    attrNameSnakeCase: true
  - pattern: 'java.lang<type=OperatingSystem><>((?!process_cpu_time)\w+):'
    name: os_$1
    type: GAUGE
    attrNameSnakeCase: true
EOF


```

> 03 修改你的java应用启动方式 添加jmx.jar 
> 注意路径和端口自己调整
- 例如kafka 
```shell script
# 在bin/kafka-server-start.sh 脚本最上面添加下面这行
export KAFKA_OPTS="-javaagent:/opt/tgzs/jmx_prometheus_javaagent-0.16.1.jar=9309:/opt/tgzs/jmx_exporter_common.yaml"

```
- 例如zookeeper
```shell script
# 在bin/zookeeper-server-start.sh 脚本EXTRA_ARGS 下面添加 EXTRA_ARGS
# EXTRA_ARGS=${EXTRA_ARGS-'-name zookeeper -loggc'}

export EXTRA_ARGS="$EXTRA_ARGS -javaagent:/opt/tgzs/jmx_prometheus_javaagent-0.16.1.jar=9142:/opt/tgzs/jmx_exporter_common.yaml"

```

- 关于jmx_exporter的配置文件可以依据不同的java app到[官网上下载使用](https://github.com/prometheus/jmx_exporter/tree/master/example_configs)




> 04 重启java服务后检查jmx_exporter的端口
- 应该有对应的端口listen
- curl 也应该有数据
```shell script

ss -ntlp |grep 9309 
curl localhost:9309 
```

> 05 将对应的采集添加到prometheus 配置中
- 注意一定要配置 java_app区分不同java应用的jmx_exporter
- 可以使用静态配置、基于文件的服务发现和其他服务发现
- job_name 最后也配置为jmx_exporter
- 静态配置如下
```yaml
- job_name: jmx_exporter
  honor_timestamps: true
  scrape_interval: 15s
  scrape_timeout: 10s
  metrics_path: /metrics
  scheme: http
  follow_redirects: true
  static_configs:
  - targets:
    - localhost:9309
    labels:
      java_app: kafka
  - targets:
    - localhost:9142
    labels:
      java_app: zookeeper
```

> 06 导入夜莺jmx_exporter内置大盘就可以看图了
