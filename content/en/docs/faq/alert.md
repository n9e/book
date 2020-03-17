---
title: "告警相关"
linkTitle: "告警相关"
weight: 2
date: 2020-03-17
description: >
  本节讲解告警相关的常见问题
---

### Q1:配置了告警策略，监控指标有异常但没有发出告警
可能是以下几种情况
1. **阈值设置有误**：查看告警策略的告警函数设置，确认告警触发条件是否满足，确认策略的生效时间是否满足
2. **策略被屏蔽**：查看告警策略屏蔽列表，确认策略是否被屏蔽
3. **通知网关问题**：到告警历史页面，查看是否已经生成的告警事件，如果有告警事件，说明是通知网关的问题
4. **策略下发问题**：
	- 执行 `curl monapi.addr/api/potal/stras/effective?all=1` 拿到全量策略列表
	- 如果列表没有此策略，查看 monapi 日志，看是否有报错信息，有报错，按照提示处理
	- 如果有，则查看judge_instance字段，找到策略分发给了哪个judge实例，登陆到judge所在机器，
	`curl 127.0.0.1:5840/api/judge/stra:id` 查看是否下发给judge
5. **judge解析策略异常**：查看judge WARNING.log 和 ERROR.log 日志，检查是否有此策略的报错信息
6. **judge没有收到数据**：修改judge日志等级为DEBUG，`tail -f DEBUG.log|grep 监控指标` 无日志输出，说明数据没有上报
