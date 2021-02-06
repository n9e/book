---
title: "插件机制"
linkTitle: "插件机制"
date: 2020-02-19
description: >
  夜莺插件机制，可以看做是agent能力的一种扩展机制，插件就是一个采集脚本，可以在页面上管理，非常灵活方便
---

菜单位置：采集配置 / 插件，大部分字段的含义一目了然不再赘述。

插件是扩展agent采集能力的一种手段，通常是一个脚本，有时是二进制，我们在web上通过配置，告诉系统哪些机器上应该执行哪些插件，执行频率是多少，插件脚本的路径在哪里等等，这个配置会下发给agent，agent就知道去哪个位置找插件，并且按照某个频率周期性去执行，*截获插件的stdout*，解析出监控指标上报。

截获的stdout要求是个json array，举例：

```json
[
    {
        "metric": "disk.io.util",
        "tags": "device=sda",
        "value": 15.4,
        "timestamp": 1554455574
    },
    {
        "metric": "api.latency",
        "tags": "api=/api/v1/auth/login,srv=n9e,mod=monapi,idc=bj",
        "value": 5.4,
        "timestamp": 1554455574
    }
]
```

还记得监控指标那一节讲解的DataModel么，这个json和那个相比，少了两个字段，一个是endpoint，一个是step，endpoint可以不为空，可以为空，为空是个偷懒的做法啦，为空的话就会默认使用agent的endpoint（一般就是本机ip），step这个字段在插件场景不需要，因为web端会配置插件多久跑一次，所以agent知道step（就是用户配置的执行频率）是多少，即使这里输出了step值，也会被agent覆盖掉。

agent执行插件，每次都是fork一个进程去跑的，就类似我们在shell环境下跑一个脚本，Linux下跑一个脚本可以给脚本传参数，可以设置环境变量，可以给脚本传入stdin，所以页面上也支持配置这些内容。有些社区同仁贡献了一些插件脚本，[在这里](https://github.com/n9e/plugin)，欢迎大家把一些好用的插件脚本贡献到这个repo。

另外，插件机制有个比较有意思的用法，可以和Prometheus社区的各类exporter做整合，之前录制过一个视频讲解这块内容，大家可以参考：[开源运维监控系统Nightingale-系列10-新版插件以及与Prometheus Exporter整合](https://www.bilibili.com/video/BV19C4y1a7zh) 
