---
title: "进程监控"
linkTitle: "进程监控"
date: 2020-02-23
description: >
  进程监控在夜莺中通过配置具体的采集策略来指定采集哪些进程，也支持读取目标机器标识文件
---


支持两种配置方式，一个是在目标机器的指定目录创建元信息文件，一个是在页面上配置采集策略。

## 目标机器元信息文件方式

进程监控和端口监控的处理逻辑类似，都是在本地touch一个文件，告诉collector去监控哪个进程，如果要监控多个进程，就需要touch多个文件。

对于端口监控而言，端口号具备唯一性，但对进程而言，进程名（查看进程名可以`cat /proc/<pid>/status`）不具备唯一性，比如python语言写的进程，进程名都是python，java语言写的进程，进程名都是java，那我们应该touch一个什么样的文件，来准确告诉collector我们要采集具体哪个进程呢？

我们需要找一个能方便区分一个进程的东西，这里我们有两个办法，一个是使用进程的名字，一个是使用进程的cmdline（参看`/proc/<pid>/cmdline`），比如touch一个10_name_n9e-monapi文件，就可以采集进程名为n9e-monapi的进程，因为n9e-monapi是go写的，进程名就是n9e-monapi，所以这么配置是OK的。如果进程name无法很好的区分，可以touch一个10_cmdline_xx的文件，假设要监控的进程pid是5648，xx就是`/proc/5648/cmdline`的一个子串即可，只要这个子串能与其他进程的cmdline区分开，就无需使用整个cmdline内容。举例：我们使用`java -c uic.properties`启动了一个java进程，其cmdline为：`java-cuic.properties`，我们要监控这个进程只需要touch一个10_cmdline_uic.properties的文件即可。

对于文件内容，和端口监控一样，可以写进程的模块名，监控数据上报的时候，就会带上这个信息，作为一个tag上报

## 页面上配置采集策略

这个方式比较简单，不过多赘述，采集策略配置的时候有个service字段，需要填写模块的名称，相当于目标机器元信息文件的内容部分