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

其中各个字段的含义参考《[DataModel](/docs/appendix/datamodel/)》一章，其中ident、alias字段也是可选的，很多情形的监控数据是没法和设备关联在一起的，比如某个域名的证书过期时间，这种监控数据就可以不传ident、alias字段。


## 数据读取

夜莺的后端存储可以支持使用M3、Prometheus、InfluxDB等等，这些存储本身就暴露了数据查询的接口，所以，大家可以直接使用存储自身提供的查询接口。也可以使用夜莺的，下面讲解一下夜莺的查询监控数据的方式。

夜莺的监控数据查询，主要是两种模式，模式一是直接传入一个promql，夜莺把这个promql透传给后端存储；模式二是指定metric、idents、tags等过滤条件，夜莺根据不同的存储后端接口，做查询条件的ETL转换。所以，如果想走夜莺的接口，建议用模式二的方式，因为模式一的方式根本不用查夜莺的接口，绕了一层多此一举，直接查后端存储即可。下面给一个模式二的查询例子：

```python
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

# python环境默认可能没有requests这个库，需要先安装一下
# 或者直接把下面的例子转换成curl命令来测试
import requests
import time

def query_data():
    now = int(time.time())
    data = {
        # 开始时间
        "start": now - 60 ,
        # 结束时间
        "end": now,
        "params": [
            {
                "metric": "system_cpu_idle",
                "idents": ["1.1.1.1", "2.2.2.2"],
                "tags": [
                    {
                        "key": "job",
                        "value": "node",
                    },
                    {
                        "key": "job",
                        "value": "db",
                    },
                    {
                        "key": "group",
                        "value": "inf",
                    },
                ],
            },
        ],
        #  返回series数量限制
        "limit": 10,
    }
    uri = 'http://localhost:8000/v1/n9e/query'
    res = requests.post(uri, json=data)
    print(res.json())

if __name__ == "__main__":
    query_data()
```

上面的例子中，核心有三个过滤条件，metric、idents、tags，相互之间是“与”的关系，tags中也有多个条件，不同key之间是“与”的关系，相同key的不同value是“或”的关系，即：`job=node|db && group=inf`

## 二次开发

夜莺的http接口主要有两个前缀，`/api/n9e`相关的是给前端JavaScript使用，用cookie做认证，写操作会有CSRF校验。`/v1/n9e`前缀的接口是给第三方系统用的。`/v1/n9e`相关的接口如何认证呢？使用token(个别接口不用token，比如数据上报查询的接口)；token从哪里找呢？去页面上，个人中心密钥里；调用接口的时候如何传递token呢？使用Authorization这个Header。比如某个第三方系统要调用夜莺的接口，先由管理员去用户中心为这个系统创建个账号，然后用这个账号登录，生成token(密钥)即可。

举例，拉取用户列表的接口，使用curl来调用，样例如下：

```bash
curl -H "Authorization: 59f668a8ac7df683a275e57a15de0db9" http://${n9e-server}/v1/n9e/users
```

## 接口安全

如果夜莺是部署在公网的，`/v1/n9e`相关的接口有些又没有认证机制，可能会被恶意利用，建议的做法：n9e-server启动的时候监听在内网ip，在n9e-server前面部署一层nginx，nginx只需对外暴露`/api/n9e`相关的接口，upstream是n9e-server的内网ip，所以别人也就访问不到了。nginx的配置可以参考《[配置nginx](/docs/appendix/nginx/)》


