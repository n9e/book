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
Return sample
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
