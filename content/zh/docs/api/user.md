---
title: "用户管理"
linkTitle: "用户管理"
weight: 4
date: 2020-03-17
description: >
  本节讲解用户管理相关API
---

### 查看用户列表
`GET /api/portal/user?limit=10&p=1`

返回样例
```json
{
  "dat": {
    "list": [
      {
        "id": 1,
        "username": "root",
        "dispname": "超管",
        "phone": "18888888888",
        "email": "",
        "im": "",
        "is_root": 1
      }
    ],
    "total": 1
  },
  "err": ""
}
```

### 新增用户
`POST /api/portal/user`

请求样例
```json
{
  "username": "nightingale",
  "password": "nightingale",
  "dispname": "nightingale",
  "phone": "18888888888",
  "email": "nightingale@didiglobal.com",
  "im": "nightingale",
}
```

### 查看用户组列表
`GET /api/portal/team?limit=10&p=1`
- mgmt 管理种类 1为管理员管理制，0为成员管理制

返回样例
```json
{
  "dat": {
    "list": [
      {
        "id": 6,
        "ident": "nightingale",
        "name": "nightingale",
        "mgmt": 0,
        "admin_objs": null,
        "member_objs": [
          {
            "id": 11,
            "username": "nightingale",
            "dispname": "nightingale",
            "phone": "18888888888",
            "email": "nightingale@didiglobal.com",
            "im": "18888888888",
            "is_root": 0
          }
        ]
      }
    ],
    "total": 1
  },
  "err": ""
}

```


### 创建用户组
`POST /api/portal/team`

请求样例
```json
{
  "ident": "nightingale",
  "name": "nightingale",
  "mgmt": 1,
  "members": [
    11
  ],
  "admins": [
    1
  ]
}
```
### 修改用户组
`PUT /api/portal/team`

请求样例
```json
{
  "ident": "nightingale",
  "name": "nightingale",
  "mgmt": 0,
  "members": [
    10
  ]
}
```





