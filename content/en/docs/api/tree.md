---
title: "Object node"
linkTitle: "Object node"
weight: 5
date: 2020-04-17
description: >
  This section explains the APIs related to object tree management
---

### View included monitoring objects
`GET /api/portal/endpoint?field=ident&limit=10&p=1`

Return to sample
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

### View objects of a node
`GET /api/portal/node/:id/endpoint?field=ident&limit=10&p=1`
- id: Node id of object tree

Return to sample
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

### Modify monitor object alias
`PUT /api/portal/endpoint/:id`
- id: Monitoring object id

Sample request
```json
{
  "alias": "n9e-node010.py"
}
```

### Mount the monitoring object under the node
`POST /api/portal/node/:id/endpoint-bind`
- id: Node id of object tree
- del_old: 1 Unhang old node 0 Keep old node

Sample request
```json
{
  "idents": [
    "10.60.2.11"
  ],
  "del_old": 0
}
```

### Unlink the monitoring object under the node
`POST /api/portal/node/:id/endpoint-unbind`
- id: Node id of object tree

Sample request
```json
{
  "idents": [
    "10.86.92.17"
  ]
}
```

### View the mount point of the monitored object
`GET /api/portal/endpoints/bindings?idents=10.86.92.17`
- idents: Monitoring object identification

Return to sample
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

### Modify the node name
`PUT /api/portal/node/21/name`
- id: Node id of object tree

Sample request
```json
{
  "name": "monapi"
}
```

### Add a child node under the node
`POST /api/portal/node`
- pid Parent node id
- leaf 1 represents a leaf node 0 represents a non-leaf node

Sample request
```json
{
  "pid": 23,
  "name": "t1",
  "leaf": 0
}
```