
---
title: "Appendix"
linkTitle: "Appendix"
weight: 10
date: 2020-04-11
description: >
  Some video tutorials, alarm sending components, etc. displayed in various chapters
---

## Video tutorial

The related video is as follows, please be sure to watch it before asking questions.

- [Nightingale's overall structure explained](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-arch-intro.mp4)
- [Quick installation](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-install-didiyun.mp4)
- [Source installation](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-install-src.mp4)
- [collector deployed separately](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-install-collector.mp4)
- [Quick Start for Nightingale Platform(1)](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-usage-001.mp4)
- [Quick Start for Nightingale Platform(2)](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-usage-002.mp4)
- [Explain address.yml](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-usage-address.mp4)
- [Explain identity Configuration](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-usage-identity.mp4)

## Alarm sending

The concept of Nightingale is that sending alarm events to redis and  no longer managed. Next, various senders read the events in redis and send them out. After all, there are too many ways to send alarms. I hope that developers in the community can work together.

Here are some typical alarm sending components for reference:

- [mail-sender](https://github.com/n9e/mail-sender)
- [wechat-sender](https://github.com/n9e/wechat-sender)
- [dingtalk-sender](https://github.com/n9e/dingtalk-sender)
