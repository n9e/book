
---
title: "Plugin"
linkTitle: "Plugin"
weight: 6
date: 2020-04-10
description: >
  We sorted out commonly used plugins in the community, especially some plugins in the Open-Falcon community that are out of repair. We guarantee that every plugin is usable and reliable.
---


The plug-in mechanism of Nightingale is similar to Open-Falcon, and the data structure is also compatible, so the plugin part can be found in [Open-Falcon](http://book.open-falcon.com/zh_0_2/usage/)，If you find that some plugins cannot be used, please contact us, we will reorganize and maintain them.

Many plugins of Open-Falcon start a process directly or use CRON driver, and then push data to open-falcon-agent. If using Nightingale, push the address to the address of the collector, ie http://127.0.0.1:2058/api/collector/push.

#### [win-collector](https://github.com/n9e/win-collector)

Collector for windows

#### [swcollector](https://github.com/gaochao1/swcollector)

Collecting switch indicators

#### [mymon](https://github.com/n9e/mymon)

For mysql monitoring

#### [jmxmon](https://github.com/toomanyopenfiles/jmxmon)

For jvm information collecting


