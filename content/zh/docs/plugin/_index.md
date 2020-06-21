
---
title: "插件"
linkTitle: "插件"
weight: 6
date: 2020-02-29
description: >
  整理社区常见插件，特别是Open-Falcon社区里的一些年久失修的插件，让每个插件都是可用的，可信赖的
---


夜莺的插件机制和Open-Falcon是类似的，数据结构也是兼容的，所以，插件部分可以参看[Open-Falcon](http://book.open-falcon.com/zh_0_2/usage/)，如果哪些插件已经年久失修，请联系我们，我们会重新整理维护起来

很多插件都是直接起一个进程或者用CRON驱动，往open-falcon-agent推送数据，换成夜莺的话推送地址改成collector的地址，即 http://127.0.0.1:2058/api/collector/push

#### [win-collector](https://github.com/n9e/win-collector)

windows版本的collector

#### [swcollector](https://github.com/gaochao1/swcollector)

采集交换机的指标

另外，夜莺通过插件机制可以整合各类Prometheus的Exporter，把Prometheus的Exporter作为夜莺的采集器，具体如何整合请参考 [夜莺新版插件机制](https://www.bilibili.com/video/BV19C4y1a7zh/)


