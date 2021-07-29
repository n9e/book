---
title: "FAQ"
linkTitle: "FAQ"
date: 2021-06-01
description: >
  一些常见的问题的解释。安装的问题、使用的问题、理念的问题，都会放到这里。
---

### FAQ001

> 户管理里面的角色 Admin Standard Guest 有啥区别？

Admin是权限最高的内置角色，不可删除，可以做所有操作；Standard是普通用户角色，大部分事情都可以干，只有用户相关的操作没有权限，比如创建用户、修改用户、删除用户等；Guest表示访客，只有查看权限，没有写权限。

另外，系统中有部分内容会有特殊的权限控制，比如告警策略分组，是可以配置对应的管理团队的，这种情况会做更精细化的控制，除了Admin之外，只有在管理团队的用户才能修改该策略分组，普通的Standard角色就没法修改、删除了。

如果想增加一个新角色，也可以，需要修改数据库表的内容，但是Admin角色在代码里很多地方Hardcode的，不要把Admin角色删除了。修改数据库里的角色要非常慎重，你要对鉴权这块逻辑非常清楚，即你要真的知道你在干什么！

### FAQ002

> ERROR alert/consume.go:211 notify: exec script ./etc/script/notify.py occur error: exit status 1

notify.py脚本里引用了一些python的库，特别是requests这个库，很可能是这个库没有安装导致的，用这个命令测试一下：`python -c "import requests"`，如果报错说 ImportError: No module named requests ，那就是这个库没安装导致的。百度一下“python install requests”

### FAQ003

> 隔离机房，如何采集到监控数据，夜莺有没有类似zabbix-proxy的组件？

夜莺是PUSH模型，n9e-server会监听两个端口，一个http端口一个rpc端口，n9e-agentd会采集监控数据，然后通过http端口推送给n9e-server。如果有某个机房网络隔离，至少要在这个隔离机房找一台转发机器，转发机器至少可以连通n9e-server，在转发机器上部署haproxy或者nginx，利用haproxy或者nginx进行流量转发。即：n9e-agentd的配置文件里，把服务端地址指向haproxy或nginx的地址，haproxy或nginx接收到请求之后转发给n9e-server。目前这个版本，haproxy和nginx只要代理n9e-server的http端口即可，无需代理rpc端口，不过后续架构可能会演进，简单起见，可以把n9e-server的两个端口都做代理。

### FAQ004

> 夜莺v5版本支持Windows的监控吗？

夜莺支持Linux、Windows、MacOS环境的监控，安装章节使用的安装脚本是CentOS环境下使用的，如要获取Windows的客户端，可以去agentd的release页面下载：[下载地址](https://github.com/n9e/n9e-agentd/releases)

### FAQ005

> 夜莺v5版本怎么没有Grafana的DataSource了？

夜莺v5拥抱各类时序数据库，底层可以支持Prometheus、M3、InfluxDB来做时序数据存储，这些存储都可以对接Grafana，所以，让Grafana直接对接这些存储就好了，夜莺不需要实现一个单独的DataSource。夜莺采集的数据会写入时序库，Grafana可以读取这些时序库，于是，就串联起来了。

### FAQ006

> 个人中心的密钥，是做什么用的？

是用来调用夜莺的接口的，具体可以参考 [API调用](/docs/appendix/api/) 章节。这个Token就是一个用户凭证，调用接口的时候带上这个Token，夜莺就知道当前是以什么身份在调用接口，是否有权限调用相关接口。
