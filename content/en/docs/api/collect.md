---
title: "Collection configuration"
linkTitle: "Collection configuration"
weight: 3
date: 2020-04-17
description: >
  This section explains the APIs related to acquisition and configuration.
---
###Field description
Support the creation of three collection strategies of `port`` proc` `log`
Field description
- nid：Associated object tree node
- name：Collection name
- tags：Collect additional reported tags
- step：Probing period
- comment：Remarks
- port Unique field
	- port：Port number detected
	- timeout: Timeout period of the probe port, in seconds
- proc Unique field
  - collect_method：There are two kinds of process collection methods: cmd and name
  - target：The target of the process collection, when cmd is selected, it is the command line, and when name is the process name

The explanation of the log collection field can be viewed on the [Log Monitoring] (../../usage/logger/) page


### Create Acquisition
`POST /api/portal/collect` 

Sample request
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
      "unit": "frequency",
      "file_path": "/home/xiaoju/alarm/log/DEBUG.log",
      "time_format": "dd/mmm/yyyy:HH:MM:SS",
      "step": 10,
      "pattern": "flush"
    }
  }
]
```
### Update collection
`PUT /api/portal/collect` 

Sample request
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
    "unit": "frequency",
    "file_path": "/home/xiaoju/alarm/log/DEBUG.log",
    "time_format": "dd/mmm/yyyy:HH:MM:SS",
    "step": 10,
    "pattern": "flush"
  }
}
```
### Delete acquisition
`DELETE /api/portal/collect` 

Sample request
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

### View the collection strategy list
`GET /api/portal/collect/list?nid=1&type=port` 
- nid：Node id of the associated object tree, optional
- type：Collection type [proc, port, log]

Return to sample
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

### View a single acquisition strategy
`GET /api/portal/collect?type=proc&id=1` 
- type：Collection type[proc,port,log]
- id：Collection strategy id
Return to sample
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