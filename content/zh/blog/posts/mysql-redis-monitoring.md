---
title: "MySQL Redis MongoDB 最简单易用的监控方案，没有之一"
linkTitle: "MySQL Redis 等基础组件监控"
slug: mysql-redis-mongo-monitoring
date: 2021-02-05
description: >
  MySQL Redis MongoDB 最简单易用的监控方案，没有之一，推荐给运维老铁
---

还在用Zabbix、Prometheus、Open-Falcon监控各类基础组件么？是否疲于寻找各类采集脚本、采集插件、Exporter？各种配置维护是否感觉心力憔悴？兄弟，我懂你，而且，我有招~

这里隆重介绍一下Nightingale（滴滴夜莺）的3.5版本，这个版本引入了一个新模块叫Prober，专门用来采集各类中间件，只需页面点点点，就可以完成监控配置，而且内置了监控大盘和告警策略（预计2021年2月初上线），一键搞定！

#### 什么原理？

其实是集成了telegraf的能力，telegraf是InfluxDB开源的一个采集器，可以采集非常多类型的中间件，比如MySQL、Redis、Mongo、ElasticSearch、RabbitMQ、Ceph、Nginx等等，具体可以查看 [这里](https://github.com/influxdata/telegraf)

telegraf采集各种中间件都是要配置一个ini格式的配置，夜莺呢，就是把ini格式的配置做了页面化，让大伙可以在页面上配置，同时对采集器做了区分region的分片，可以同时承载非常大量的采集任务，某个采集器挂掉，也可以自动摘除，有可靠的高可用保障。

#### 效果如何？

耳听为虚眼见为实，截几张图给大家看看效果，入口在采集配置下面的组件部分：

![](https://s3-gz01.didistatic.com/n9e-pub/n9e-book/post/post-01-01.png)

上图我测试了4种组件采集，由于这个prober的扩展非常容易，开源社区有人贡献了Nginx、ElasticSearch、Prometheus的采集（刚开源Prober一周，就有朋友新贡献了3个组件采集器，可见集成telegraf，与夜莺做整合是非常容易的），所以如果大家拉取 https://github.com/didi/nightingale master上面的代码编译monapi和prober，应该不但可以看到mysql redis mongodb github这4类，还可以看到nginx、elasticsearch、prometheus的选项。

下面我们看一个mongodb的采集配置页面，主要就是填写mongodb的连接地址，夜莺的Prober模块就会周期性的去连接到mongodb实例，运行采集命令收集数据，简单到爆！

![](https://s3-gz01.didistatic.com/n9e-pub/n9e-book/post/post-01-02.png)

#### 查看数据？

采集到的数据在即时看图里可以非常方便的查看，如下图所示：

![](https://s3-gz01.didistatic.com/n9e-pub/n9e-book/post/post-01-03.png)

当然，既然有监控数据了，自然就可以配置监控大盘，最近我们刚刚上线了内置监控大盘功能，当前只是做了 `linux_host` 相关的基础指标大盘，后面会把各类中间件的大盘也内置到系统里，一键导入创建。

#### 详细资料？

上周我们请Prober的作者喻波做了一次在线分享，分享回放在这里：[https://www.bilibili.com/video/BV1hz4y1S7rD/](https://www.bilibili.com/video/BV1hz4y1S7rD/) 




