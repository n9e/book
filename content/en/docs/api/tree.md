---
title: "对象节点"
linkTitle: "对象节点"
weight: 5
date: 2020-03-17
description: >
  本节讲解对象树管理相关API
---

### 查看已收录的监控对象
`GET /api/portal/endpoint?field=ident&limit=10&p=1`

返回样例
```json
{
  "dat": {
    "list": [
      {
        "id": 8,
        "ident": "10.60.2.10",
        "alias": "n9e-node010.py"
      },
      {
        "id": 9,
        "ident": "10.60.2.11",
        "alias": "n9e-node011.py"
      }
    ],
    "total": 2
  },
  "err": ""
}
```

### 查看某个节点的对象
`GET /api/portal/node/:id/endpoint?field=ident&limit=10&p=1`
- id: 对象树的节点id

返回样例
```json
{
  "dat": {
    "list": [
      {
        "id": 19,
        "ident": "10.86.92.17",
        "alias": ""
      }
    ],
    "total": 1
  },
  "err": ""
}
```

### 修改监控对象别名
`PUT /api/portal/endpoint/:id`
- id: 监控对象的id

请求样例
```json
{
  "alias": "n9e-node010.py"
}
```

### 节点下挂载监控对象
`POST /api/portal/node/:id/endpoint-bind`
- id: 对象树的节点id
- del_old: 1解挂旧节点 0保留旧节点

请求样例
```json
{
  "idents": [
    "10.60.2.11"
  ],
  "del_old": 0
}
```

### 节点下解挂监控对象
`POST /api/portal/node/:id/endpoint-bind`
- id: 对象树的节点id

请求样例
```json
{
  "idents": [
    "10.86.92.17"
  ]
}
```

### 查看监控对象的挂载点
`GET /api/portal/endpoints/bindings?idents=10.86.92.17`
- idents: 监控对象标识

返回样例
```json
{
  "dat": [
    {
      "ident": "10.86.92.17",
      "alias": "",
      "nodes": [
        {
          "id": 21,
          "pid": 19,
          "name": "monapi",
          "path": "mon.monapi",
          "leaf": 1,
          "note": ""
        },
        {
          "id": 28,
          "pid": 19,
          "name": "judge",
          "path": "mon.judge",
          "leaf": 1,
          "note": ""
        }
      ]
    }
  ],
  "err": ""
}
```

### 修改节点名称
`PUT /api/portal/node/21/name`
- id: 对象树的节点id

请求样例
```json
{
  "name": "monapi"
}
```

### 节点下添加子节点
`POST /api/portal/node`
- pid 父节点id
- leaf 1表示为叶子节点 0表示非叶子节点

请求样例
```json
{
  "pid": 23,
  "name": "t1",
  "leaf": 0
}
```