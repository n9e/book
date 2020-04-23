---
title: "Monitoring indicators "
linkTitle: "Monitoring indicators"
date: 2020-04-22
description: >
 Here is a brief introduction to the data structure describing the monitoring indicators. Nightingale and Open-Falcon are compatible
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

Among them, metric is the name of the monitoring indicator, endpoint is the monitoring entity, tags are the attribute labels of the monitoring data, step is the reporting period of the monitoring data, value is the current value of the monitoring indicator, timestamp is the current timestamp, and the unit is seconds.The counterType field indicates the indicator type, and supports GAUGE and COUNTER. If this field is not reported, the default is GAUGE. If the reported indicator is of COUNTER type, it needs to be specified explicitly.In addition, the calculation logic of the COUNTER type index is implemented in the collector module, so do not push the COUNTER type index directly to transfer.

In addition, if the tags field does not want to use the above string concatenation method, you can use the tagsMap field instead. The above two indicators are rewritten as follows using tagsMap:

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
