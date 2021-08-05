---
title: "日志监控"
linkTitle: "日志监控"
date: 2021-05-24
description: >
  夜莺内置的日志监控，并非是把日志收集到中心在中心处理，而是使用了一种更轻量的方式，在服务端配置正则表达式，下发到agentd，agentd会流式读取日志，然后从日志中根据正则提取metrics数据，所以本质上，这仍然是metrics监控的一种扩展机制。如果大家需要中心化的日志处理解决方案，可以参考我们的logi产品。
---

日志采集支持本地配置文件(在 conf.d/log.d 目录下)和web页面两种配置方式，下面以本地配置文件举例来说明日志采集如何配置，如果想采集nginx日志中 /api/n9e/push 被访问的次数，本地配置文件如下

```yaml
instances:
# @param name - string - required
# name 在页面上是采集名称，已log_开头，是日志采集监控指标的metric
- name: log_api_count

# @param file_path - string - required
# file_path 要采集的日志文件所在的位置，支持通配符（*），例如 /var/log/myapp/*.log、/var/log/myapp/*/*.log
  file_path: /var/log/nginx/access.log

# param pattern -  string - required
# pattern 表示要匹配的日志的关键字，支持正则
  pattern: /api/n9e/push

# param tags_pattern -  map[string]string - optional
# tags_pattern 是从日志提取监控指标tag的配置，是一个map，key是tag的key,
# value是一个正则表达式，正则中匹配到的子串是会赋值给value，下面配置是将日志中的状态码提取出来作为tag
  tags_pattern:
    code: HTTP\/1.1"\s([0-9]{3})

# #param func -  string - required - enum: count, histogram
# func是匹配到日志之后的计算方式，支持count和histogram
# count 表示统计每个周期匹配到的日志的个数
# histogram 会产生5个指标，$name_avg $name_max $name_median $name_count $name_95percentile，采集周期内的统计值
# 注意使用 histogram 时，主正则（pattern）要可以提取数字，因为需要计算统计值
  func: count

# @param tags - list of strings - optional
# A list of tags to attach to every metric and service check emitted by this instance.
# Learn more about tagging at https://docs.datadoghq.com/tagging
# 附件标签，如果想给监控指标打一个属于某个服务的tag，可以配置如下
# tags:
#   - service:n9e
```

页面功能各字段与配置文件基本一一对应，此处不过多赘述。另外，web上有一个配置验证的字段，可以放一条真实日志看正则是否能够正常匹配。