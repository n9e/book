---
title: "Monitoring data"
linkTitle: "Monitoring data"
weight: 1
date: 2020-04-17
description: >
  This section explains the API related to monitoring data.
---

### Report data
 Nightingale's transfer and collector modules both provide interfaces for reporting data 
local collector report API `POST http://127.0.0.1:2058/api/collector/push`   
center transfer report API `POST /api/transfer/push`      
For the meaning of the fields, see [Data Specification](https://n9e.didiyun.com/docs/usage/metric/)   

Sample request 
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

### Query metric
`POST /api/index/metric`

Sample request
```json
{
    "endpoints": ["host1","host2"]
}
```
Return to sample
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

### Query tags
`POST /api/index/tagkv`

Sample request
```json
{
    "endpoints": ["host1","host2"],
    "metrics": ["disk.used.percent"],
}
```
Return to sample
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
### Query monitoring data
`POST /api/transfer/data`

Sample request
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
Return sample
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
