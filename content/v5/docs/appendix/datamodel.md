---
title: "DataModel"
linkTitle: "DataModel"
date: 2021-06-25
description: >
  介绍夜莺监控数据的格式，对于理解何为一条监控指标非常重要，对于未来自行采集监控数据上报给夜莺非常重要，对于理解告警策略的过滤规则非常重要！
---

首先举个例子：

```json
{
    "ident": "10.4.5.6",
    "alias": "c3-cloud-ceph01.bj",
    "metric": "system_fs_inodes_used",
    "tags": {
        "device": "/dev/vda1"
    },
    "time": 1624062385,
    "value": 351
}
```

> “资源”一词的解释：资源即某个监控实体，比如某个机器、交换机、防火墙、调制解调器等等。夜莺提供了资源分组的能力，用于应对传统的物理机、虚拟机的服务混部监控场景，ident和alias字段就是用于标识资源

- ident：资源唯一标识，如果是机器的监控指标，这个字段大都填充为机器ip，如果是跟设备无关的指标，这个字段可以为空
- alias：资源名称，如果是机器，这个字段很可能是填充为hostname，当然，如果是设备无关的指标，这个字段也是空，即使是设备的指标，这个字段也允许为空
- metric：监控指标的名字
- tags：监控数据的维度信息，上例中从监控指标的名字可以得知代表inode的使用量，但是一个机器有多个盘，具体是哪个盘还需要表达出来，这种就用tags来描述
- time：UNIX时间戳，单位是秒
- value：监控数据此刻的值，只能是数值，服务端最终都转换为浮点数处理

夜莺的服务端提供了监控数据上报的接口，如果是要把监控数据上报给服务端，就用上例中的数据结构组装为list上报即可，比如：

```bash
curl -X POST -H "Content-Type: application/json" http://n9e-server-address/v1/n9e/push -d '[
    {
        "ident": "10.4.5.6",
        "alias": "c3-cloud-ceph01.bj",
        "metric": "system_fs_inodes_used",
        "tags": {
            "device": "/dev/vda1"
        },
        "time": 1624062385,
        "value: 351
    },
    {
        "ident": "10.4.5.6",
        "alias": "c3-cloud-ceph01.bj",
        "metric": "system_fs_inodes_used",
        "tags": {
            "device": "/dev/vda2"
        },
        "time": 1624062385,
        "value: 35109
    }
]'
```

另外，夜莺的客户端即n9e-agentd进程支持statsd的方式接收数据，在agentd的配置文件中将statsd功能enable即可，statsd的介绍在：[github](https://github.com/statsd/statsd) ，statsd是一个指标聚合神器，通常用于应用监控的场景。使用node.js语言开发，n9e-agentd是go写的，即用go实现了node.js的逻辑，statsd社区提供的各种语言的sdk都可以直接使用，业务方用这些sdk埋点，把数据推送地址设置为n9e-agentd的udp地址，即可与夜莺打通，用夜莺来做应用监控。

