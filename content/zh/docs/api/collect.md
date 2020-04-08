---
title: "采集配置"
linkTitle: "采集配置"
weight: 3
date: 2020-03-17
description: >
  本节讲解采集配置相关API
---
### 字段说明
支持创建 `port` `proc` `log` 三种采集策略
字段说明
- nid：关联的对象树节点
- name：采集名称
- tags：采集额外上报的tags
- step：探测的周期
- comment：备注
- port 独有字段
	- port：探测的端口端口号
	- timeout: 探测端口的超时时间，单位为秒
- proc 独有字段
  - collect_method：进程采集方式有 cmd 和 name 两种 
  - target：进程采集的目标，选cmd时为命令行，name时为进程名

log采集字段解释到[日志监控](../../usage/logger/)页面查看


### 创建采集
`POST /api/portal/collect` 

请求样例
```json
[
  {
    "type": "port",
    "data": {
      "nid": 2,
      "collect_type": "port",
      "name": "service.port",
      "tags": "",
      "port": 90,
      "timeout": 3,
      "step": 20,
      "comment": ""
    }
  },
  {
    "type": "proc",
    "data": {
      "nid": 2,
      "collect_type": "proc",
      "name": "service",
      "collect_method": "cmd",
      "target": "tsdb",
      "tags": "",
      "step": 20,
      "comment": ""
    }
  },
  {
    "type": "log",
    "data": {
      "nid": 2,
      "collect_type": "log",
      "func_type": "FLOW",
      "name": "LOG.aa",
      "func": "cnt",
      "unit": "次数",
      "file_path": "/home/xiaoju/alarm/log/DEBUG.log",
      "time_format": "dd/mmm/yyyy:HH:MM:SS",
      "step": 10,
      "pattern": "flush"
    }
  }
]
```
### 更新采集
`PUT /api/portal/collect` 

请求样例
```json
//proc
{
  "type": "proc",
  "data": {
    "id": 1,
    "nid": 2,
    "collect_type": "proc",
    "name": "444444444",
    "step": 20,
    "comment": "test",
    "target": "tsdb",
    "collect_method": "cmd"
  }
}
//port
{
  "type": "port",
  "data": {
    "id": 1,
    "nid": 2,
    "collect_type": "port",
    "name": "33333",
    "step": 10,
    "comment": "bbbb2222aa",
    "port": 900,
    "timeout": 3
  }
}
//log
{
  "type": "log",
  "data": {
    "nid": 2,
    "collect_type": "log",
    "func_type": "FLOW",
    "name": "LOG.aa",
    "func": "cnt",
    "unit": "次数",
    "file_path": "/home/xiaoju/alarm/log/DEBUG.log",
    "time_format": "dd/mmm/yyyy:HH:MM:SS",
    "step": 10,
    "pattern": "flush"
  }
}
```
### 删除采集
`DELETE /api/portal/collect` 

请求样例
```json
[
  {
    "type": "port",
    "ids": [1,2]
  },
  {
    "type": "proc",
    "ids": [1,2]
  }
]
```

### 查看采集策略列表
`GET /api/portal/collect/list?nid=1&type=port` 
- nid：关联的对象树节点id，选填
- type：采集类型[proc,port,log]

返回样例
```json
{
  "dat": [
    {
      "id": 4,
      "nid": 2,
      "collect_type": "port",
      "name": "tsdb",
      "tags": "service=tsdb",
      "step": 10,
      "comment": "",
      "creator": "root",
      "created": "2019-09-03T18:24:02+08:00",
      "last_updator": "root",
      "last_updated": "2019-09-03T18:24:02+08:00",
      "port": 8046,
      "timeout": 3
    },
    {
      "id": 13,
      "nid": 2,
      "collect_type": "log",
      "name": "log.a",
      "step": 10,
      "comment": "",
      "creator": "root",
      "created": "2019-09-17T18:26:06+08:00",
      "last_updator": "root",
      "last_updated": "2019-09-17T18:26:06+08:00",
      "tags": null,
      "file_path": "a",
      "time_format": "dd/mmm/yyyy:HH:MM:SS",
      "pattern": "a",
      "func": "cnt",
      "func_type": "FLOW",
      "unit": "",
      "degree": 0,
      "zerofill": 0,
    }
  ],
  "err": ""
}
```

### 查看单个采集策略
`GET /api/portal/collect?type=proc&id=1` 
- type：采集类型[proc,port,log]
- id：采集策略id
返回样例
```json
{
  "dat": {
    "id": 3,
    "nid": 21,
    "collect_type": "port",
    "name": "monapi",
    "tags": "service=monapi",
    "step": 10,
    "comment": "",
    "creator": "root",
    "created": "2020-02-21T23:24:33+08:00",
    "last_updator": "root",
    "last_updated": "2020-02-21T23:24:33+08:00",
    "port": 8058,
    "timeout": 3
  },
  "err": ""
}
```