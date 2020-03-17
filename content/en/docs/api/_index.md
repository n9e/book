---
title: "API"
linkTitle: "API"
weight: 9
date: 2020-03-17
description: >
  本节会介绍Nightingale对外提供的API
---
### 前言
1. 所有接口的错误返回均为以下格式，错误信息会写入到`err`字段中
```json
{
    "dat": {},
    "err": "error msg"
}
```

2. `/api/portal`相关接口会进行鉴权，需要在header中增加认证字段
```
key为：Authorization
value为： username:password
```

3. Nightingale各个模块前面会有一个nginx，所以所有接口的地址默认为nginx的地址