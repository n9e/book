---
title: "监控指标"
linkTitle: "监控指标"
date: 2020-02-29
description: >
  这里简单介绍一下描述监控指标的数据结构，Nightingale和Open-Falcon是保持兼容的
---


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

其中，metric是监控指标名称，endpoint是监控实体，tags是监控数据的属性标签，step为监控数据的上报周期，value是监控指标的当前值，timestamp是当前时间戳，单位是秒。干掉了counterType字段，统一使用GAUGE类型，不再支持COUNTER和DERIVE类型。

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
