
---
title: "附录"
linkTitle: "附录"
weight: 10
date: 2020-03-15
description: >
  一些散置在各个章节的视频教程、告警发送组件等
---

## 视频教程

相关视频如下，请一定要看完...

- [夜莺整体架构详解](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-arch-intro.mp4)
- [快速安装演示](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-install-didiyun.mp4)
- [夜莺平台快速入门（一）](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-usage-001.mp4)
- [夜莺平台快速入门（二）](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-usage-002.mp4)
- [解释address.yml](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-usage-address.mp4)
- [解释identity配置](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-usage-identity.mp4)

## 告警发送

Nightingale的理念，是将告警事件扔到redis里就不管了，接下来由各种sender来读取redis里的事件并发送，毕竟发送报警的方式太多了，适配起来比较费劲，希望社区同仁能够共建。

这里提供一些典型的告警发送组件，供参考：

- [mail-sender](https://github.com/n9e/mail-sender)
- [wechat-sender](https://github.com/n9e/wechat-sender)
