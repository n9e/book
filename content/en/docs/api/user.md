---
title: "User Management"
linkTitle: "User Management"
weight: 4
date: 2020-04-17
description: >
 This section explains user management related APIs
---

### View user list
`GET /api/portal/user?limit=10&p=1`

Return to sample
```json
{
  "dat": {
    "list": [
      {
        "id": 1,
        "username": "root",
        "dispname": "root",
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

### New users
`POST /api/portal/user`

Sample request
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

### View list of user groups
`GET /api/portal/team?limit=10&p=1`
- mgmt Management type 1 is the administrator management system, 0 is the member management system

Return to sample
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


### Create User Group
`POST /api/portal/team`

Sample request
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
### Modify user group
`PUT /api/portal/team`

Sample request
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