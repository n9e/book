---
title: "组件采集，中心化的高可用的集群采集器：Prober"
linkTitle: "组件采集"
date: 2020-02-18
description: >
  夜莺在3.5.0版本引入了一个新组件叫Prober，作为一个中心化的采集器，可以采集MySQL、Redis、MongoDB等组件的监控数据
---

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

Prober的作者喻波曾做了一次在线分享，分享回放在这里：[https://www.bilibili.com/video/BV1hz4y1S7rD/](https://www.bilibili.com/video/BV1hz4y1S7rD/) 



