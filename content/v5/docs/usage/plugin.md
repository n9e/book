---
title: "插件监控"
linkTitle: "插件监控"
date: 2021-05-25
description: >
  夜莺插件机制，可以看做是agentd能力的一种扩展机制，插件通常来说，就是一个采集脚本(也可以是个二进制，只要具备可执行权限即可)，可以在页面上控制哪些机器要执行插件，非常灵活方便
---

插件是扩展agentd采集能力的一种手段，通常是一个脚本，有时是二进制，我们在web上通过配置，告诉夜莺哪些机器上应该执行哪些插件，执行频率是多少，插件脚本的路径在哪里等等，这个配置会下发给agentd，agentd就知道去哪个位置找插件，并且按照某个频率周期性去执行，截获插件的stdout，解析出监控指标上报。

截获的stdout要求是个json array，举例：

```bash
[
    {
        "metric": "disk.io.util",
        "tags": {
            "device": "sda"
        },
        "value": 15.4,
        "type": "gauge"
    },
    {
        "metric": "api.latency",
        "tags": {
            "api": "/v1/login",
            "service": "n9e",
            "module": "monapi",
            "region": "bj"
        },
        "value": 5.4,
        "type": "gauge"
    }
]
```

这个数据结构可以对照《[DataModel](/docs/appendix/datamodel/)》一章的内容来看，上面的插件输出样例其实没有ident、alias、time字段，因为插件是agentd驱动的，所以agentd会自动补上这些字段，无需插件输出这些字段。

相比《[DataModel](/docs/appendix/datamodel/)》一章的介绍，上例中多了一个type字段，这个字段是插件机制特有的，type可取的类型有3种：gauge、increase、rate。下面简单解释下这3种类型的意义和场景：

- gauge: 原值类型，插件输出什么值，agentd都不做处理，原封不动发给server
- increase: 表示增量，agentd会把本周期取到的插件的值与上一周期的值做差值
- rate: 表示速率，agentd会把本周期取到的插件的值与上一周期的值做差值，然后除以周期时间

大家可以把下面这个插件拿去执行一下做个测试，对这三种类型的理解会更深，场景上来说，其中gauge用的较多，另外两个类型很少用到

```bash
#!/bin/sh

now=$(date +%s)

echo '[
    {
        "metric": "plugin_example_gauge",
        "tags": {
            "type": "testcase",
            "author": "ulric"
        },
        "value": '${now}',
        "type": "gauge"
    },
    {
        "metric": "plugin_example_rate",
        "tags": {
            "type": "testcase",
            "author": "ulric"
        },
        "value": '${now}',
        "type": "rate"
    },
    {
        "metric": "plugin_example_increase",
        "tags": {
            "type": "testcase",
            "author": "ulric"
        },
        "value": '${now}',
        "type": "increase"
    }
]'
```

agentd执行插件，每次都是fork一个进程去跑的，就类似我们在shell环境下跑一个脚本，Linux下跑一个脚本可以给脚本传参数，可以设置环境变量，可以给脚本传入stdin，所以页面上也支持配置这些内容。

欢迎大家写文章介绍自己的插件，大家共建生态！

