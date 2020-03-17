---
title: "监控数据"
linkTitle: "监控数据"
weight: 1
date: 2020-03-17
description: >
  本节讲解监控数据相关API
---

### 上报数据
Nightingale的transfer和collector两个模块都提供了上报数据的接口    
本地collector上报接口 `POST http://127.0.0.1:2058/api/collector/push`   
中心transfer上报接口 `POST /api/transfer/push`      
字段含义见[数据规范](https://n9e.didiyun.com/docs/usage/metric/)   

请求样例  
```json
[
    {
        "metric":"cpu.util",
        "endpoint":"192.168.1.2",
        "timestamp":1559733442,
        "step":10,
        "value":1,
        "tags":""
    }
]
```

### 查询metric
`POST /api/index/metric`

请求样例
```json
{
    "endpoints": ["host1","host2"]
}
```
返回样例
```json
{
    "dat": [
        {
            "metrics": [
                "cpu.idle"
            ],
        }
    ],
    "err": "",
}
```

### 查询tags
`POST /api/index/tagkv`

请求样例
```json
{
    "endpoints": ["host1","host2"],
    "metrics": ["disk.used.percent"],
}
```
返回样例
```json
{
    "dat": [
        {
            "endpoints": ["host1","host2"],
            "metric": "disk.used.percent",
            "tagkv": [
                {
                    "tagk": "mount",       
                    "tagv": ["/", "/home"]
                },
            ]
        }
    ],
    "err":""
}
```
### 查询监控数据
`POST /api/transfer/v2/data`

请求样例
```json
{
  "start": 1562925134,
  "end": 1562925234,
  "series": [
    {
      "endpoints": [
        "127.0.0.1",
        "127.0.0.2"
      ],
      "metric": "proc.num",
      "tagkv": [
        {
          "tagk": "target",
          "tagv": [
            "collector"
          ]
        },
        {
          "tagk": "service",
          "tagv": [
            "n9e-collector"
          ]
        }
      ]
    }
  ]
}
```
返回样例
```json
{
    "dat": [
        {
            "start": 1562925134,
            "end": 1562925234,
            "endpoint": "127.0.0.1",
            "counter": "proc.num/service=n9e-collector,target=collector",
            "step": 10,
            "values": [
                {
                    "timestamp": 1562925120,
                    "value": 1
                }
            ]
        },
        {
            "start": 1562925134,
            "end": 1562925234,
            "endpoint": "127.0.0.2",
            "counter": "proc.num/service=n9e-collector,target=collector",
            "step": 10,
            "values": [
                {
                    "timestamp": 1562925210,
                    "value": 0
                },
                {
                    "timestamp": 1562925220,
                    "value": 0
                },
                {
                    "timestamp": 1562925230,
                    "value": 0
                }
            ]
        }
    ],
    "err": ""
}
```