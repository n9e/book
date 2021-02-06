---
title: "变更检查"
linkTitle: "变更检查"
date: 2020-03-06
description: >
  本节讲解各个模块的变更checklist
---

## 前言

可以通过几个方面判断变更是否异常：

1. 判断更改之后进程是否存活，此方式可以通过给模块添加进程监控来实现，当然，更直接的是登录机器查看
2. 检查各个模块的日志，包括ERROR、WARNING这种落到文件中的日志，以及进程的标准输出，进程如果是用control脚本启动的，标准输出在stdout文件里，如果是用systemd启动的，就要用systemctl status查看了，或者使用journalctl
3. 通过观察模块的业务指标是否正常，本节重点介绍下各个模块需要关注哪些业务指标
    
Nightingale的各个组件启动之后，会将自身的的一些状态数据上报到监控系统，通过上报的指标可以观测其运行是否正常，下面介绍下各个模块的监控指标
## transfer 变更
transfer的核心业务指标如下，统计周期为10s

| 监控指标        | 含义   |
| --------   | ----- |
| n9e.transfer.points.in     | 接收点数| 
| n9e.transfer.points.out.tsdb        |向tsdb发送点数| 
| n9e.transfer.points.out.tsdb.err        |向tsdb发送失败的点数| 
| n9e.transfer.points.out.judge        |向judge发送的点数| 
| n9e.transfer.points.out.judge.err        |向juege发送失败的点数| 
| n9e.transfer.stra.count        |获取监控策略条数| 

如果transfer变更之后，transfer集群上述指标的数据没有明显的变化，则说明变更符合预期

## tsdb 变更
tsdb的核心业务指标如下，统计周期为10s

| 监控指标        | 含义   |
| --------   | ----- |
| n9e.tsdb.points.in     | 接收的点数| 
| n9e.tsdb.query.miss     | 查询数据为空的次数| 
| n9e.tsdb.index.out.err     | 推送索引失败条数| 

如果tsdb变更之后，tsdb集群上述指标的数据没有明显的变化，则说明变更符合预期

## index 变更
index的核心业务指标如下，统计周期为10s

| 监控指标        | 含义   |
| --------   | ----- |
| n9e.index.query.counter.miss     |fullmatch接口查索引未命中次数| 
| n9e.index.xclude.miss     |xclude接口查索引未命中次数| 

如果index变更之后，index集群上述指标的数据没有明显的变化，则说明变更符合预期

## judge 变更
judge的核心业务指标如下，统计周期为10s

| 监控指标        | 含义   |
| --------   | ----- |
| n9e.judge.push.in     |接收点数| 
| n9e.judge.running     |正在执行的judge任务数| 
| n9e.judge.stra.count     |获取的策略数| 

如果judge变更之后，judge集群上述指标的数据没有明显的变化，则说明变更符合预期
     