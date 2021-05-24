---
title: "进程监控"
linkTitle: "进程监控"
date: 2020-02-23
description: >
  介绍如何在夜莺系统里做进程监控
---


菜单位置：采集配置 / 进程，大部分字段的含义和“端口监控”雷同，不再赘述。这里着重说明进程监控的特殊的点。

进程监控核心是在监控进程的数量，比如n9e-job进程，进程活着，名字是n9e-job的进程数肯定就是1，进程死了，名字是n9e-job的进程数就变成0了，此时就应该报警。但是，进程名字（查看进程名可以`cat /proc/<pid>/status`，有个name字段）有时不具备唯一性，导致我们没法统计进程数量，比如python语言写的进程，进程名都是python，java语言写的进程，进程名都是java，那我们怎么来准确告诉agent我们要采集具体哪个进程呢？

我们需要找一个能方便区分标识进程的东西，这里我们有两个办法，一个是使用进程的名字，一个是使用进程的cmdline（参看`/proc/<pid>/cmdline`）。页面配置有个字段叫“采集方式”就是为此设计，如果是C、C++、Go这种语言写的程序，一般就用进程名来监控即可，如果是PHP、Python、Java这种语言写的程序，就要选择采集方式为：命令行，那命令行这个字段怎么填？假设要监控某个Java进程，pid是5648，命令行就填为`/proc/5648/cmdline`的一个子串即可，只要这个子串能与其他进程的cmdline区分开，就无需使用整个cmdline内容。举例：我们使用`java -c uic.properties`启动了一个java进程，其cmdline为：`java-cuic.properties`，我们要监控这个进程只需要把命令行设置为`uic.properties`即可，`uic.properties`是5648这个进程的cmdline的一个子串，而且可以和其他进程区分开。

v3版本的进程监控和v2版本一样，可以参考 [视频教程](https://www.bilibili.com/video/BV1dQ4y1N7Aq/)



