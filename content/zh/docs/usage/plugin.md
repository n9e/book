---
title: "插件机制"
linkTitle: "插件机制"
date: 2020-02-19
description: >
  夜莺可以指定某个目录为插件目录，读取该目录下符合特定格式的文件，当成插件执行，以此扩展collector的能力
---

> update: 注意：新版本插件执行需要在页面（采集配置页面）上配置才能运行，不是把脚本扔到plugin目录就可以啦！
> 好处：
> - 支持给插件传参数，传环境变量，传Stdin，非常灵活
> - 分发插件的时候可以无差别分发到所有机器，不同机器执行不同的插件直接在web上即可控制
> - 详见[开源运维监控系统Nightingale-系列10-新版插件以及与Prometheus Exporter整合] (https://www.bilibili.com/video/BV19C4y1a7zh)

collector组件虽然内置了很多监控指标的采集，但无法解决所有场景，所以提供了插件机制，以便扩展collector的功能。

查看collector的配置文件，里边配置了plugin的目录，我们只需把插件脚本放到这个目录下，collector就会自动探测到，然后周期性运行。对于业务系统的监控指标采集，可以把采集脚本放到业务程序发布包中，随着业务代码上线而上线（上线的时候把脚本放到collector的plugin目录下），随着业务代码升级而升级，随着业务代码的下线而下线（下线的时候把脚本从collector的plugin目录下移除），这样会比较容易管理。

对于插件，有如下几个要求：

- 插件脚本必须具有可执行权限，部署完了脚本记得chmod +x一下
- 插件脚本可以是sh、py、pl、rb，甚至可以是二进制，只要机器上有runtime环境
- 插件脚本的命名：`${step}_xx.xx`，比如`20_uptime.sh`，`${step}`是在告诉collector多久跑一次
- plugin目录下非`${step}_xx.xx`命名格式的文件或者目录可以存在没关系，collector不会识别为插件
- 插件执行之后要在stdout输出一个json array，collector会截获这个输出，解析为监控指标上报
- 如果插件执行报错了，报错消息要打印到stderr，不要打印到stdout

下面给一个shell编写的插件例子20_uptime.sh：

```bash
#!/bin/bash

duration=$(cat /proc/uptime | awk '{print $1}')
localip=$(ifconfig `route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|awk -F ':' '{print $NF}'|head -n 1)
step=$(basename $0|awk -F'_' '{print $1}')
echo '[
    {
        "endpoint": "'${localip}'",
        "tags": "",
        "timestamp": '$(date +%s)',
        "metric": "sys.uptime.duration",
        "value": '${duration}',
        "step": '${step}'
    }
]'
```

下面给一个python编写的插件例子20_plugin_status.py：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import time
import commands
import json
import sys
import os

items = []


def collect_myself_status():
    item = {}
    item["metric"] = "plugin.myself.status"
    item["value"] = 1
    item["tags"] = ""
    items.append(item)


def main():
    code, endpoint = commands.getstatusoutput(
        "timeout 1 ifconfig `route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|awk -F ':' '{print $NF}'|head -n 1")
    if code != 0:
        sys.stderr.write('cannot get local ip')
        return

    timestamp = int("%d" % time.time())
    plugin_name = os.path.basename(sys.argv[0])
    step = int(plugin_name.split("_", 1)[0])

    collect_myself_status()

    for item in items:
        item["endpoint"] = endpoint
        item["timestamp"] = timestamp
        item["step"] = step

    print json.dumps(items)


if __name__ == "__main__":
    main()
```

另外，Open-Falcon的社区里可以看到很多第三方指标采集器都不是以插件机制存在，都是以独立daemon进程或者cron脚本的方式，这种方式和插件类似，只是需要自己将采集的监控数据push给collector或者transfer，也算是一种广义的插件。
