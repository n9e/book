---
title: "监控指标"
linkTitle: "监控指标"
date: 2020-02-29
description: >
  这里简单介绍一下描述监控指标的数据结构，Nightingale和Open-Falcon是保持兼容的
---

这里简单介绍一下描述监控指标的数据结构，Nightingale和Open-Falcon是保持兼容的

```json
[
    {
        "metric": "disk.io.util",
        "endpoint": "10.86.12.13",
        "tags": "device=sda",
        "value": 15.4,
        "timestamp": 1554455574,
        "step": 20
    },
    {
        "metric": "api.latency",
        "endpoint": "10.86.12.13",
        "tags": "api=/api/v1/auth/login,srv=n9e,mod=monapi,idc=bj",
        "value": 5.4,
        "timestamp": 1554455574,
        "step": 20
    }
]
```

其中，metric是监控指标名称，endpoint是监控实体，tags是监控数据的属性标签，step为监控数据的上报周期，value是监控指标的当前值，timestamp是当前时间戳，单位是秒。counterType字段表示指标类型，支持GAUGE、COUNTER、SUBTRACT，如果不上报这个字段，默认为GAUGE，如果上报的指标是COUNTER或SUBTRACT类型，需要明确指定，另外，COUNTER和SUBTRACT类型的指标的计算逻辑是在agent模块实现的，所以，不要把COUNTER或SUBTRACT类型的指标直接推给transfer，只能推给agent。

另外，tags字段如果不想使用上面的字符串拼接方式，可以使用tagsMap字段来代替，上面两条指标使用tagsMap改写如下：

```json
[
    {
        "metric": "disk.io.util",
        "endpoint": "10.86.12.13",
        "tagsMap": {
            "device": "sda"
        },
        "value": 15.4,
        "timestamp": 1554455574,
        "step": 20
    },
    {
        "metric": "api.latency",
        "endpoint": "10.86.12.13",
        "tagsMap": {
            "api": "/api/v1/auth/login",
            "srv": "n9e",
            "mod": "monapi",
            "idc": "bj"
        },
        "value": 5.4,
        "timestamp": 1554455574,
        "step": 20
    }
]
```

上面的数据结构用来描述机器相关的指标，是比较合适的，但有时会遇到一些指标并非依托某个机器实体存在，即实在是不知道endpoint字段填什么合适，比如某个ceph集群的整体存储使用率，某个域名的访问成功率，为了解决这类问题，夜莺把监控指标分成两类，“设备相关”的和“设备无关”的（v3版本之后引入了RDB模块，即用户资源中心，允许各类资源注册到系统里，除了主机设备之外，rds实例、redis实例、vip等，都可以注册进来，所以，“设备相关”和“设备无关”，应该改成“资源相关”和“资源无关”可能更贴切）。“设备相关”的，就是endpoint字段有值的，“设备无关”的，就是endpoint字段没有值的，相当于：设备无关的监控指标不属于任何实体设备，即全局可见。

但是，不同的部门可能会上报相同metric导致没法区分，所以“设备无关”的监控指标，要求在上报的时候填充一个nid字段（即endpoint字段为空，nid字段不为空，注意，nid字段也设计为了字符串类型），即，这个“设备无关”的监控指标，是和某个树（组织资源树）节点绑定在一起的，nid在这里，一定程度起到了namespace的作用。

举个例子，n9e这个服务的monapi模块，可能部署了多个机器，要看这个模块整体的平均延迟，需要做一次指标聚合计算，把这多个机器的数据聚合在一起，生成一个新指标：`aggr.api.latency.ms`，这个指标的endpoint就没法写成某个具体的机器了，这种场景就可以设计为“设备无关”的监控指标。

```json
[
    {
        "metric": "aggr.api.latency.ms",
        "nid": "162",
        "tagsMap": {
            "api": "/api/v1/auth/login",
            "srv": "n9e",
            "mod": "monapi",
            "idc": "bj"
        },
        "value": 5.4,
        "timestamp": 1554455574,
        "step": 20
    }
]
```


