
---
title: "介绍"
linkTitle: "介绍"
weight: 2
date: 2020-02-29
description: >
  对夜莺做一个简单直观的展示，讲解与Open-Falcon的异同，讲解其架构和后续发展
---

## Nightingale介绍

本节对Nightingale做一个简要介绍，让您对系统有个初步直观感受，为后面的深入了解做准备。

### Nightingale系统截图

直接看系统截图，是最直观的了解系统的方式，这里截几张图，让大家有个初步的认识

![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-tree.png)

上图是节点下对象页面，对象，是指目标监控对象，通常是指机器，这棵树称为服务树，从上到下描述为组织架构关系、产品服务模块关系、机房和机器挂载关系，此树支持灵活定制。这本质上是对机器的一个分组机制，实际做监控策略配置的时候，必然是不同用途的机器配置不同的告警策略，所以，一个灵活的机器分组机制，非常重要

![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-stra.png)

上图是监控策略配置页面，列表中的策略都是应用到didi.storage节点的，故而，该节点下的所有子节点挂载的所有的机器都会应用这个策略，任何一台机器触发相关阈值都会告警，是不是超方便 :-)

![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-screen.png)

监控大盘的定制做了大幅易用性改进，支持了图表阈值，支持了图表分类，新增图表和排序管理都是可见即所得的方式，巡检大盘的定制从此不再是困难。


### 与Open-Falcon的异同

由于Nightingale与Open-Falcon有很深的血缘关系，想必大家对二者的对比会较为感兴趣。总体来看，理念上有很强的延续性，架构和产品体验上的改进巨大。

#### 与Open-Falcon的不同点

- 告警引擎重构为推拉结合模式，通过推模式保证大部分策略判断的效率，通过拉模式支持了nodata告警，去除原来的nodata组件，简化系统部署难度
- 引入了服务树，对endpoint进行层级管理，去除原来扁平的HostGroup，同时干掉告警模板，告警策略直接与服务树节点绑定，大幅提升灵活度和易用性
- 干掉原来的基于数据库的索引库，改成内存模式，单独抽出一个index模块处理索引，避免了原来MySQL单表达到亿级的尴尬局面，索引基于内存之后效率也大幅提升
- 存储模块Graph，引入facebook的Gorilla，即内存TSDB，近期几个小时的数据默认存内存，大幅提升数据查询效率，硬盘存储方式仍然使用rrdtool
- 告警引擎judge模块通过心跳机制做到了故障自动摘除，再也不用担心单个judge挂掉导致部分策略失效的问题，index模块也是采用类似方式保证可用性
- 客户端中内置了日志匹配指标抽取能力，web页面上支持了日志匹配规则的配置，同时也支持读取目标机器特定目录下的配置文件的方式，让业务指标监控更为易用
- 将portal(falcon-plus中的api)、uic、dashboard、hbs、alarm合并为一个模块：monapi，简化了系统整体部署难度，原来的部分模块间调用变成进程内方法调用，稳定性更好

#### 与Open-Falcon的相同点

- 数据模型没有变化，仍然是metric、endpoint、tags的组织方式，所以agent基本是可以复用的，Nightingale中的agent叫collector，融合了原来Open-Falcon的agent和falcon-log-agent的逻辑，各种监控插件也都是可以复用的
- 数据流向和整体处理逻辑是类似的，仍然使用灵活的推模型，分为数据存储和告警判断两条链路，下一节我们基于架构图详细展开

### Nightingale架构概述

![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-arch.png)

- collector即agent，可以采集机器常见指标，支持日志监控，支持插件机制，支持业务通过接口直接上报数据
- transfer提供rpc接口接收collector上报的数据，然后通过一致性哈希，将数据转发给多台tsdb和多台judge
- tsdb即原来的graph组件，用于存储历史数据，支持配置为双写模式提升系统容灾能力，tsdb会把监控数据转发一份给index
- index是索引模块，替换原来的mysql方案，在内存里构建索引，便于后续数据检索，性能大幅提升
- judge是告警引擎，从monapi(portal)同步监控策略，然后对接收到的数据做告警判断，如满足阈值，则生成告警事件推到redis
- monapi(alarm)从redis读取judge生成的事件，进行二次处理，补充一些元信息，生成告警消息，重新推回redis
- 各发送组件，比如mail-sender、sms-sender等，从redis读取告警消息，发送告警，抽出各类sender是为了后续定制方便
- monapi集成了原来多个模块的功能，提供接口给js调用，api前缀为`/api/portal`，数据查询走transfer，干掉了原来的query组件，api前缀为`/api/transfer`，索引查询的api前缀`/api/index`，于是，前面搭建nginx，即可通过不同location将请求转发到不同后端
- 数据库仍然使用mysql，主要存储的内容包括：用户信息、团队信息、树节点信息、告警策略、监控大盘、屏蔽策略、采集策略、部分组件心跳信息等

*如果想了解更细节的内容，可以参看这个小视频：[夜莺架构讲解](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-arch-intro.mp4)*

### Nightingale后续发展

1、提供监控指标聚合组件，现在的架构可以解决机器级、模块级的监控，但是集群维度的监控指标，是需要聚合整个集群的所有模块、机器的指标，做一些加和、求平均之类的操作，这个滴滴内部已经有相关组件，但是整理出来还需要时间

2、接入容器监控，与Prometheus结合，或者直接采集cadvisor的数据，这块具体如何操作暂未想明白，还需再推敲一下

3、完善更多监控插件，之前Open-Falcon社区里的很多插件都是可以直接用的，我们会尽量补充社区没有的插件，对于社区里已经年久失修的插件，我们会二次整理，让Nightingale周边更完善