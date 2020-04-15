---
title: "告警相关"
linkTitle: "告警相关"
weight: 2
date: 2020-03-17
description: >
  本节讲解告警策略相关API
---

### 字段说明
- name：策略名称
- nid: 策略关联的对象树节点id
- excl_nid: 排除关联对象树节点下的子节点id
- tags: 监控指标的tags
- priority: 告警等级，可以设置1，2，3
- alert_dur: 告警统计周期，单位为秒
- enable_stime：策略生效开始时间
- enable_etime：策略生效终止时间
- enable_days_of_week：策略生效日期
- exprs
  - eopt：操作符，枚举[=,!=,>,>=,<,<=]
  - func：告警函数，支持all happen max min avg sum diff pdiff nodata
  - metric：监控指标
  - params：告警函数需要的参数
  - threshold：告警函数需要的阈值
- recovery_dur：持续多少秒则产生恢复event，0表示立即产生恢复event
- recovery_notify：0 发送恢复通知 1不发送恢复通知
- converge：告警通知收敛，第1个值表示收敛周期，单位秒，第2个值表示周期内允许发送告警次数
- notify_group：告警信息接收组
- notify_user：告警信息接收人
- callback：告警触发之后的回调地址
- need_upgrade：是否配置告警升级 0表示否 1表示是
- alert_upgrade
  - duration： 告警持续多久触发升级，单位为秒
  - level：升级的告警等级
  - users： 升级之后发送的告警信息接收人
  - groups： 升级之后发送的告警信息接收组

### 创建策略
`POST /api/portal/stra`

请求样例
```json
{
  "name": "all必触发", 
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
返回样例
```json
{
  "err":"",
  "dat": {"id": 1}
}
```

### 更新策略
`PUT /api/portal/stra`

请求样例
```json
{
  "id":1,
  "name": "all必触发", 
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
返回样例
```json
{
  "err":"",
  "dat":"ok"
}
```

### 删除策略
`DELETE /api/portal/stra`

请求样例
```json
{
    "ids":[4]
}
```
返回样例
```json
{
  "err":"",
  "dat":"ok"
}
```

### 查看所有策略
`GET /api/portal/stra?nid=1`   
nid：服务树节点id，选填，不填则获取所有策略

返回样例
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

### 查看单个策略
`GET /api/portal/stra/:sid`

返回样例
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

### 查看所有生效策略
`GET /api/portal/stra/effective`

返回样例
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

### 查看未恢复告警历史
`GET /api/portal/event/cur?limit=10&nodepath=mon.monapi&p=1&stime=1584427852&etime=1584435052`   
参数说明
- limit 返回个数
- p 页数
- nodepath 对象树节点
- priorities 告警级别
- stime 筛选范围起始时间
- etime 筛选范围终止时间

返回样例
```json
{
  "dat": {
    "list": [
      {
        "id": 16,
        "sid": 14,
        "sname": "某磁盘无法正常读写",
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
          "已收敛"
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

### 查看全部告警历史
`GET /api/portal/event/his?limit=10&nodepath=mon.monapi&p=1&stime=1584427852&etime=1584435052`   
参数说明
- limit 返回个数
- p 页数
- nodepath 对象树节点
- priorities 告警级别
- stime 筛选范围起始时间
- etime 筛选范围终止时间

返回样例
```json
{
  "dat": {
    "list": [
      {
        "id": 16,
        "sid": 14,
        "sname": "某磁盘无法正常读写",
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
          "已收敛"
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

### 创建告警屏蔽
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

### 解除告警屏蔽
`DELETE /api/portal/maskconf/:id` 
- id：屏蔽配置id  

### 查看告警屏蔽
`GET /api/portal/node/:id/maskconf`   

返回样例
```json
{
  "dat": [
    {
      "id": 6,
      "nid": 28,
      "node_path": "didi.mon.judge",
      "metric": "cpu.idle",
      "tags": "",
      "cause": "快速屏蔽",
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

