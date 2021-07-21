---
title: "JMX监控"
linkTitle: "JMX监控"
date: 2021-05-17
description: >
  JMX用于采集JVM的一些监控指标，对于Java类的一些中间件，一般会用这种方式，Java写的业务进程不推荐用JMX方式，建议直接使用micrometer这种SDK埋点，micrometer会自动采集JVM相关监控指标。
---

我们使用prometheus生态的jmx_exporter来采集监控数据。[夜莺介绍](/docs/intro/)章节，介绍了集成exporter的原理。假设我们使用prometheus作为夜莺的时序存储库，只需要在prometheus里配置jmx_exporter的抓取规则即可。如果我们使用M3或InfluxDB作为时序库，那就需要搞一个prometheus作为纯粹的抓取器，抓取jmx_exporter的数据，然后通过remote write方式，写入M3或InfluxDB。不管是哪种方式，只要数据进了时序库，夜莺就可以消费了，可以在夜莺中查看监控数据，配置告警规则，查看告警事件，管理告警接收人等。

下面以kafka和zookeeper为例，介绍通过jmx采集监控数据的做法。对于prometheus生态的其他exporter，道理都一样，都是想方设法把exporter的数据抓取到时序库。然后交由夜莺服务端消费。

### 01. 下载jar包

下载jar包，姑且把jar包放到/opt/tgzs，目录可以调，但是所有的地方要保持一致

```bash
mkdir -p /opt/tgzs
cd /opt/tgzs
wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.16.1/jmx_prometheus_javaagent-0.16.1.jar
```

### 02. 准备配置 

准备jmx_exporter基础配置文件，也是姑且放在/opt/tgzs目录，可以调，但是其他地方要对应上。下面的配置可以导出一些机器指标：

```bash
cd /opt/tgzs
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

### 03. 引用jar包和配置

JVM启动的时候，有个`-javaagent`的参数，可以把jmx的jar包引入，下面以kafka和zookeeper举例。jar包和yaml文件的路径以及JMX端口自行调整，特别是端口，别跟现有机器上的进程冲突了即可。

下面的是kafka的示例：

```bash
# 在bin/kafka-server-start.sh 脚本最上面添加下面这行
export KAFKA_OPTS="-javaagent:/opt/tgzs/jmx_prometheus_javaagent-0.16.1.jar=9309:/opt/tgzs/jmx_exporter_common.yaml"
```

下面是zookeeper的示例：

```bash
# 在bin/zookeeper-server-start.sh 脚本EXTRA_ARGS 下面添加 EXTRA_ARGS
# EXTRA_ARGS=${EXTRA_ARGS-'-name zookeeper -loggc'}
export EXTRA_ARGS="$EXTRA_ARGS -javaagent:/opt/tgzs/jmx_prometheus_javaagent-0.16.1.jar=9142:/opt/tgzs/jmx_exporter_common.yaml"
```

关于jmx_exporter的配置文件可以依据不同的中间件到[官网](https://github.com/prometheus/jmx_exporter/tree/master/example_configs)下载使用，比如tomcat、weblogic、activemq、cassandra、flink、spark，都有提供对应的yaml配置。

### 04. 重启Java中间件

重启Java中间件，对应的JMX端口应该会自动监听，curl这个端口，status code应该可以返回200，可以通过下面的命令测试：

```bash
# 以9309端口举例
ss -ntlp | grep 9309 
curl localhost:9309 
```

### 05. 配置抓取器

前面几步做完，Java中间件已经监听了JMX的端口，后面就可以使用prometheus的抓取能力来抓取这些JMX端口的监控数据。注意一定要配置java_app这个label，后面导入的内置大盘，就是依赖这个label来区分不同的中间件。job_name默认就设置为jmx_exporter即可。静态抓取配置如下：

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

### 06. 导入大盘

在夜莺的监控大盘页面，选择导入内置大盘，导入jmx_exporter即可查看jmx相关监控数据。
