---
title: "绘图相关"
linkTitle: "绘图相关"
weight: 1
date: 2020-03-17
description: >
  本节讲解采集&看图相关的常见问题
---
### Q1:监控指标已发送到服务端，但最终看不到图
此情况一般是数据格式不正确，或者数据并没有上报到服务端，假设发送的指标为 n9e.points.in
1. 首先看下transfer和tsdb模块是否有 n9e.points.in 的错误日志   
登陆到transfer部署的机器，执行 `tail -f /home/n9e/logs/transfer/WARNING.log|grep n9e.points.in`   
登陆到tsdb部署的机器，执行 `tail -f /home/n9e/logs/tsdb/WARNING.log|grep n9e.points.in`
2. 如果没有报错信息，再次确认监控指标是否上报到了服务端，修改transfer的日志等级为 DEBUG   
执行 `tail -f /home/n9e/logs/transfer/DEBUG.log|grep n9e.points.in` 如果没有日志出现，说明没有并没有上报到服务端，排查发送端的问题

### Q2:监控插件放到指定目录了，但最终看不到图
1. 一般遇到这种问题都是插件执行失败了，或者插件上报的数据格式不正确，可以登陆到有问题的机器，查看 /home/n9e/logs/collector/ERROR.log 有没有插件相关的错误日志，如有，解决之。如果没有错误日志，按照[Q1](#q1)思路排查

### Q3:自己写程序，按照[数据规范](../usage/metric/)向collector或者transfer上报数据，但看不到图 
1. 首先将程序的request body和resp body 都打印出来，查看 resp 中 err 不为空说明上报格式有问题，request body为空说明没有上报数据
2. 如果上报正常，按照[Q1](#q1)思路排查

### Q4:配置了日志、进程或者端口采集，但看不到相关监控图
1. 查看采集策略是否已经下发到目标机器的collector       
`curl 目标机器IP:2058/api/collector/stra`
2. 如果可以查到配置的采集，tail -f /home/n9e/logs/collector/ERROR.log 查看是否有错误日志
3. 有报错则按照日志提示去处理，如果没有报错，按照[Q1](#q1)思路排查