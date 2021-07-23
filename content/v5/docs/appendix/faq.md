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
