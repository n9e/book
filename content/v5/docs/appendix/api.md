---
title: "API调用"
linkTitle: "API调用"
date: 2021-06-21
description: >
  介绍夜莺对外暴露的API，用于自行采集监控数据上报、读取监控数据分析、对夜莺页面做二次开发等场景。重点讲解监控数据上报、读取的接口，二次开发相关的接口，在http下的router.go里可以找到所有接口，会简略介绍，需要大家自行翻阅代码。
---

## 数据上报

直接看使用curl命令上报数据的例子即可，如果是其他语言翻译一下就完了。监控指标组织为json list格式，放到http request body中，post给n9e-server即可。

```bash
curl -X POST -H "Content-Type: application/json" http://n9e-server-address/v1/n9e/push -d '[
    {
        "ident": "10.4.5.6",
        "alias": "c3-cloud-ceph01.bj",
        "metric": "system_fs_inodes_used",
        "tags": {
            "device": "/dev/vda1"
        },
        "time": 1624062385,
        "value: 351
    },
    {
        "ident": "10.4.5.6",
        "alias": "c3-cloud-ceph01.bj",
        "metric": "system_fs_inodes_used",
        "tags": {
            "device": "/dev/vda2"
        },
        "time": 1624062385,
        "value: 35109
    }
]'
```


## 数据读取

TODO:


## 二次开发

夜莺的http接口主要有两个前缀，`/api/n9e`相关的是给前端JavaScript使用，用cookie做认证，写操作会有CSRF校验。`/v1/n9e`前缀的接口是给第三方系统用的。`/v1/n9e`相关的接口如何认证呢？使用token(个别接口不用token，比如数据上报查询的接口)；token从哪里找呢？去页面上，个人中心密钥里；调用接口的时候如何传递token呢？使用Authorization这个Header。比如某个第三方系统要调用夜莺的接口，先由管理员去用户中心为这个系统创建个账号，然后用这个账号登录，生成token(密钥)即可。

举例，拉取用户列表的接口，使用curl来调用，样例如下：

```bash
curl -H "Authorization: token-from-web-gen" http://${n9e-server}/v1/n9e/users
```

## 接口安全

如果夜莺是部署在公网的，`/v1/n9e`相关的接口有些又没有认证机制，可能会被恶意利用，建议的做法：n9e-server启动的时候监听在内网ip，在n9e-server前面部署一层nginx，nginx只需对外暴露`/api/n9e`相关的接口，upstream是n9e-server的内网ip，所以别人也就访问不到了。
