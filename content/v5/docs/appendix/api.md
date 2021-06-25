---
title: "API"
linkTitle: "API"
date: 2021-06-26
description: >
  介绍夜莺对外暴露的API，用于自行采集监控数据上报、读取监控数据分析、对夜莺页面做二次开发等场景。重点讲解监控数据上报、读取的接口，二次开发相关的接口，在http下的router.go里可以找到所有接口，会简略介绍，需要大家自行翻阅代码。
---

## 数据上报

TODO:


## 数据读取

TODO:


## 二次开发

夜莺的http接口主要有两个前缀，`/api/n9e`相关的是给前端JavaScript使用，用cookie做认证，写操作会有CSRF校验。`/v1/n9e`前缀的接口是给第三方系统用的。`/v1/n9e`相关的接口如何认证呢？使用token；token从哪里找呢？去页面上，个人中心密钥里；调用接口的时候如何传递token呢？使用Authorization这个Header。比如某个第三方系统要调用夜莺的接口，先由管理员去用户中心为这个系统创建个账号，然后用这个账号登录，生成token(密钥)即可。

举例，拉取用户列表的接口，使用curl来调用，样例如下：

```bash
curl -H "Authorization: token-from-web-gen" http://${n9e-server}/v1/n9e/users
```
