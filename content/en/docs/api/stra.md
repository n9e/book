---
title: "Alarm related"
linkTitle: "Alarm related"
weight: 2
date: 2020-04-17
description: >
  This section explains the API related to the alarm strategy
---

### Field description
- name：Strategy name
- nid: The node id of the object tree associated with the policy
- excl_nid: Exclude the child node id under the node of the associated object tree
- tags: Tags for monitoring indicators
- priority: Alarm level, you can set 1, 2, 3
- alert_dur: Alarm statistics period, in seconds
- enable_stime：Policy effective start time
- enable_etime：End time of effective policy
- enable_days_of_week：Strategy Effective Date
- exprs
  - eopt：Operator, enumeration [=,! =,>,> =, <, <=]
  - func：Alarm function, support all happen max min avg sum diff pdiff nodata
  - metric：Monitoring indicators
  - params：Parameters required by the alarm function
  - threshold：The threshold required by the alarm function
- recovery_dur：How many seconds the recovery event will be generated, 0 means that recovery event will be generated immediately
- recovery_notify：0 Send recovery notification 1 Do not send recovery notification
- converge：Alarm notification convergence, the first value indicates the convergence period, in seconds, and the second value indicates the number of alarms allowed to be sent during the period
- notify_group：Alarm information receiving group
- notify_user：Alarm information recipient
- callback：Callback address after the alarm is triggered
- need_upgrade：Whether to configure alarm escalation 0 means no 1 means yes
- alert_upgrade
  - duration： How long the alarm lasts to trigger the upgrade, in seconds
  - level：Upgraded alarm level
  - users： Recipient of alarm information sent after upgrade
  - groups： Receive group of alarm information sent after upgrade

### Create Strategy
`POST /api/portal/stra`

Sample request
```json
{
  "name": "all Must trigger", 
  "nid": 21,
  "excl_nid": null,    
  "priority": 3,
  "alert_dur": 60,    
  "exprs": [
    {	
      "eopt": "!=",
      "func": "all",
      "metric": "cpu.idle",
      "params": [],
      "threshold": 0
    }
  ],
  "tags": [],
  "recovery_dur": 0,
  "recovery_notify": 1,
  "alert_upgrade": {
    "duration": 60,
    "level": 1,
    "users": [],
    "groups": []
  },
  "converge": [3600,1],
  "notify_group": [],
  "notify_user": [5],
  "callback": "",
  "enable_stime": "00:00",
  "enable_etime": "23:59",
  "enable_days_of_week": [0,1,2,3,4,5,6],
  "need_upgrade": 0,
  "id": 13
}
```
Return to sample
```json
{
  "err":"",
  "dat": {"id": 1}
}
```

### Update strategy
`PUT /api/portal/stra`

Sample request
```json
{
  "id":1,
  "name": "all must triger", 
  "nid": 21,
  "excl_nid": null,    
  "priority": 3,
  "alert_dur": 60,    
  "exprs": [
    { 
      "eopt": "!=",
      "func": "all",
      "metric": "cpu.idle",
      "params": [],
      "threshold": 0
    }
  ],
  "tags": [],
  "recovery_dur": 0,
  "recovery_notify": 1,
  "alert_upgrade": {
    "duration": 60,
    "level": 1,
    "users": [],
    "groups": []
  },
  "converge": [3600,1],
  "notify_group": [],
  "notify_user": [5],
  "callback": "",
  "enable_stime": "00:00",
  "enable_etime": "23:59",
  "enable_days_of_week": [0,1,2,3,4,5,6],
  "need_upgrade": 0,
  "id": 13
}
```
Return to sample
```json
{
  "err":"",
  "dat":"ok"
}
```

### Delete strategy
`DELETE /api/portal/stra`

Sample request
```json
{
    "ids":[4]
}
```
Return to sample
```json
{
  "err":"",
  "dat":"ok"
}
```

### View all strategies
`GET /api/portal/stra?nid=1`   
nid：Service tree node id, optional, if not filled, get all strategies

Return to sample
```json
{
  "dat": [
    {
      "id": 1,
      "name": "io.util is greater than 90%",
      "category": 1,
      "nid": 100,
      "alert_dur": 600,
      "recovery_dur": 120,
      "enable_stime": "00:00",
      "enable_etime": "23:59",
      "priority": 3,
      "callback": "",
      "creator": "root",
      "created": "2019-03-06T16:47:16+08:00",
      "last_updator": "root",
      "last_updated": "2019-03-06T16:47:16+08:00",
      "excl_nid": [99],
      "exprs": [
        {
          "eopt": ">",
          "func": "abs",
          "metric": "qps",
          "params": [3],
          "threshold": 10
        }
      ],
      "tags": [
        {
          "tkey": "host",
          "topt": "=",
          "tval": ["nightingale.host1"]
        }
      ],
      "enable_days_of_week": [0,1,2,3,4,5,6],
      "converge": [60,3],
      "recovery_notify": 1,
      "notify_group": [1,3],
      "notify_user": [1,3],
      "leaf_nids": null,
      "need_upgrade":1,
      "alert_upgrade":{
        "users":[1,3],
        "groups":[1,3],
        "duration":1000,
        "level":1
      }
    },
  ],
  "err": ""
}
```

### View a single strategy
`GET /api/portal/stra/:sid`

Return to sample
```json
{
  "dat":
    {
      "id": 1,
      "name": "io.util大于90%",
      "category": 1,
      "nid": 100,
      "alert_dur": 600,
      "recovery_dur": 120,
      "enable_stime": "00:00",
      "enable_etime": "23:59",
      "priority": 3,
      "callback": "",
      "creator": "root",
      "created": "2019-03-06T16:47:16+08:00",
      "last_updator": "root",
      "last_updated": "2019-03-06T16:47:16+08:00",
      "excl_nid": [99],
      "exprs": [
        {
          "eopt": ">",
          "func": "abs",
          "metric": "qps",
          "params": [3],
          "threshold": 10
        }
      ],
      "tags": [
        {
          "tkey": "host",
          "topt": "=",
          "tval": ["nightingale.host1"]
        }
      ],
      "enable_days_of_week": [0,1,2,3,4,5,6],
      "converge": [60,3],
      "recovery_notify": 1,
      "notify_group": [1,3],
      "notify_user": [1,3],
      "leaf_nids": null,
      "need_upgrade":1,
      "alert_upgrade":{
        "users":[1,3],
        "groups":[1,3],
        "duration":1000,
        "level":1
      }
    },
  "err": ""
}
```

### View all effective strategies
`GET /api/portal/stra/effective`

Return to sample
```json
{
  "dat": [
    {
      "id": 1,
      "name": "io.util大于90%",
      "category": 1,
      "nid": 100,
      "alert_dur": 600,
      "recovery_dur": 120,
      "enable_stime": "00:00",
      "enable_etime": "23:59",
      "priority": 3,
      "callback": "",
      "creator": "root",
      "created": "2019-03-06T16:47:16+08:00",
      "last_updator": "root",
      "last_updated": "2019-03-06T16:47:16+08:00",
      "excl_nid": [99],
      "exprs": [
        {
          "eopt": ">",
          "func": "abs",
          "metric": "qps",
          "params": [3],
          "threshold": 10
        }
      ],
      "tags": [
        {
          "tkey": "host",
          "topt": "=",
          "tval": ["nightingale.host1"]
        }
      ],
      "enable_days_of_week": [0,1,2,3,4,5,6],
      "converge": [60,3],
      "recovery_notify": 1,
      "notify_group": [1,3],
      "notify_user": [1,3],
      "leaf_nids": null,
      "need_upgrade":1,
      "alert_upgrade":{
        "users":[1,3],
        "groups":[1,3],
        "duration":1000,
        "level":1
      }
    },
  ],
  "err": ""
}
```

### View unrecovered alarm history
`GET /api/portal/event/cur?limit=10&nodepath=mon.monapi&p=1&stime=1584427852&etime=1584435052`   
Parameter Description
-limit returns the number
-p pages
-nodepath object tree node
-priorities alarm level
-stime filter range start time
-etime filtering range end time

Return to sample
```json
{
  "dat": {
    "list": [
      {
        "id": 16,
        "sid": 14,
        "sname": "A disk cannot be read and written normally",
        "node_path": "mon.monapi",
        "nid": 21,
        "endpoint": "192.168.1.2",
        "priority": 1,
        "event_type": "alert",
        "category": 1,
        "hashid": 775538174592195020,
        "etime": 1584435048,
        "value": "disk.rw.error: 3",
        "info": " disk.rw.error(all,60s) > 0",
        "tags": "mount=/home",
        "created": "2020-02-21T23:32:31+08:00",
        "detail": [
          {
            "metric": "disk.rw.error",
            "tags": {
              "mount": "/home"
            },
            "points": [
              {
                "timestamp": 1584435048,
                "value": 3
              },
              {
                "timestamp": 1584435028,
                "value": 3
              },
              {
                "timestamp": 1584435008,
                "value": 3
              }
            ]
          }
        ],
        "users": [
          "qinyening"
        ],
        "groups": [],
        "status": [
         "Converged"
        ],
        "claimants": [
          "qinyening",
          "root"
        ],
        "need_upgrade": 0,
        "alert_upgrade": {
          "users": null,
          "groups": [],
          "duration": 60,
          "level": 1
        }
      }
    ],
    "total": 1
  },
  "err": ""
}
```

### View all alarm history
`GET /api/portal/event/his?limit=10&nodepath=mon.monapi&p=1&stime=1584427852&etime=1584435052`   
Parameter Description
-limit returns the number
-p pages
-nodepath object tree node
-priorities alarm level
-stime filter range start time
-etime filtering range end time

Return to sample
```json
{
  "dat": {
    "list": [
      {
        "id": 16,
        "sid": 14,
        "sname": "A disk cannot be read and written normally ",
        "node_path": "mon.monapi",
        "nid": 21,
        "endpoint": "192.168.1.2",
        "priority": 1,
        "event_type": "alert",
        "category": 1,
        "hashid": 775538174592195020,
        "etime": 1584435048,
        "value": "disk.rw.error: 3",
        "info": " disk.rw.error(all,60s) > 0",
        "tags": "mount=/home",
        "created": "2020-02-21T23:32:31+08:00",
        "detail": [
          {
            "metric": "disk.rw.error",
            "tags": {
              "mount": "/home"
            },
            "points": [
              {
                "timestamp": 1584435048,
                "value": 3
              },
              {
                "timestamp": 1584435028,
                "value": 3
              },
              {
                "timestamp": 1584435008,
                "value": 3
              }
            ]
          }
        ],
        "users": [
          "qinyening"
        ],
        "groups": [],
        "status": [
         "Converged"
        ],
        "claimants": [
          "qinyening",
          "root"
        ],
        "need_upgrade": 0,
        "alert_upgrade": {
          "users": null,
          "groups": [],
          "duration": 60,
          "level": 1
        }
      }
    ],
    "total": 1
  },
  "err": ""
}
```

### Create alarm mask
`POST /api/portal/maskconf`   
请求样例
```json
{
  "btime": 1584436015,
  "etime": 1584439615,
  "cause": "快速屏蔽",
  "metric": "cpu.idle",
  "endpoints": [
    "192.168.1.2"
  ],
  "nid": 28
}
```

### Unmask the alarm
`DELETE /api/portal/maskconf/:id` 
- id：屏蔽配置id  

### View alarm mask
`GET /api/portal/node/:id/maskconf`   

Return to sample
```json
{
  "dat": [
    {
      "id": 6,
      "nid": 28,
      "node_path": "didi.mon.judge",
      "metric": "cpu.idle",
      "tags": "",
      "cause": "Quick shield ",
      "user": "qinyening",
      "btime": 1584436521,
      "etime": 1584440121,
      "endpoints": [
        "192.1681.2"
      ]
    }
  ],
  "err": ""
}
```

