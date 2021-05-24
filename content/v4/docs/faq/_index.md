
---
title: "常见问题"
linkTitle: "常见问题"
weight: 9
date: 2020-03-09
description: >
  夜莺经常遇到的一些问题
---

- 喜马拉雅音频：[https://www.ximalaya.com/keji/45095827/](https://www.ximalaya.com/keji/45095827/)
- B站视频教程：[https://space.bilibili.com/442531657](https://space.bilibili.com/442531657)
- 二次开发教程：[https://xie.infoq.cn/article/30d37e98fbe52ff2a79fc04b4](https://xie.infoq.cn/article/30d37e98fbe52ff2a79fc04b4)
- 右侧二维码，扫描加小助手的好友，小助手拉你进互助交流群

#### 机器或agent挂了，nodata策略不告警

大概率是机器时间不对，agent所在机器的时间和夜莺服务端部署的机器时间不一致导致的，要用ntp同步一下

#### 如何做高可用部署

找多台机器，每台机器都部署所有服务端模块，修改address.yml，把各个addresses字段填充为实际的多台机器的IP；transfer的配置文件里，配置多个tsdb，当然了，我们更推荐大家使用 [M3DB](/docs/install/m3db/)；nginx.conf里各个upstream要配置多个实例。齐活。

