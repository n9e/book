---
title: "监控数据"
linkTitle: "监控数据"
weight: 1
date: 2020-03-17
description: >
  本节讲解监控数据相关API
---

## 上报监控数据

Nightingale的transfer和agent两个模块都提供了上报数据的接口：  

- 本地agent上报接口： `POST http://${agent.addr}/v1/push`
- 中心transfer上报接口： `POST http://${transfer.addr}/api/transfer/push`

字段含义见[数据规范](/docs/usage/metric/)。说明一下 `extra` 字段，用户可以在这个字段中添加额外的信息，比如traceId（千万不要把traceId作为tag上报，会把索引模块打爆的），当监控数据触发告警后，告警引擎会将 `extra` 的内容写到告警事件中，透传给告警历史页面，用户可以根据 `extra` 中的信息更快的定位问题。

**数据上报给agent和transfer的区别是什么？**

数据结构中有一个字段叫counterType，可以设置为：GAUGE COUNTER SUBTRACT，其中 COUNTER 和 SUBTRACT 这两种数据类型，只有agent才支持，原理是：agent在接收到数据之后，判断counterType，如果发现是 COUNTER 或者 SUBTRACT ，会自动进行计算，将结果作为GAUGE类型上报给transfer，transfer只支持GAUGE类型。如果counterType这个字段为空，会被当成GAUGE类型。

**COUNTER详解**

COUNTER通常用于采集一直递增的值，比如网卡的丢包量，从操作系统的计数器获取的数据来看，是一直递增的，即操作系统启动以来总的丢包量，显然，我们对这个值当前是多少并不关注，我们更关注的是过去一段时间，每秒丢包量是多少。于是COUNTER类型应运而生，假设采集频率是10秒，10秒之前采集到的值是v1，当时的时间戳是t1，现在采集到的最新值v2，现在的时间戳是t2，agent就会自动计算：`(v2-v1)/(t2-t1)`，得到的结果作为GAUGE上报。

**SUBTRACT详解**

SUBTRACT和COUNTER非常类似，就是在计算的时候，不除以时间差，只是计算：`v2-v1`，即最近一次统计周期内的增量是多少。

上报监控数据时采用POST方法，将监控数据组装为一个json array放到request body中，请求body样例：

```json
[
    {
        "metric":"cpu.util",
        "endpoint":"192.168.1.2",
        "timestamp":1559733442,
        "step":10,
        "value":1,
        "tags":"",
        "extra":""
    }
]
```


## 查询监控数据

`POST /api/transfer/data`

counters字段比较重要，额外讲解一下。counters是个数组，每一个元素是metric+tags的一个组合，格式是：`$metric/$tagkey1=$tagval1,$tagkey2=tagval2`，metric+tags唯一标识了一个series，即监控图上的一条线。

请求body样例：

```json
[
    {
        "start": 1598338900,
        "end": 1598338940,
        "endpoints": ["10.254.226.221"],
        "counters": ["cpu.idle"],
        "step": 20,
        "dstype": "GAUGE"
    },
    {
        "start": 1598338900,
        "end": 1598338940,
        "endpoints": ["10.254.88.96"],
        "counters": ["disk.bytes.used.percent/mount=/"],
        "step": 20,
        "dstype": "GAUGE"
    }
]
```

返回样例：

```json
{
  "dat": [
    {
      "start": 1598338900,
      "end": 1598338940,
      "endpoint": "10.254.226.221",
      "counter": "cpu.idle",
      "dstype": "GAUGE",
      "step": 20,
      "values": [
        {
          "timestamp": 1598338900,
          "value": 98.648649
        },
        {
          "timestamp": 1598338920,
          "value": 98.316498
        },
        {
          "timestamp": 1598338940,
          "value": 98.307953
        }
      ]
    },
    {
      "start": 1598338900,
      "end": 1598338940,
      "endpoint": "10.254.88.96",
      "counter": "disk.bytes.used.percent/mount=/",
      "dstype": "GAUGE",
      "step": 20,
      "values": [
        {
          "timestamp": 1598338900,
          "value": 14.636725
        },
        {
          "timestamp": 1598338920,
          "value": 14.636725
        },
        {
          "timestamp": 1598338940,
          "value": 14.63682
        }
      ]
    }
  ],
  "err": ""
}
```
