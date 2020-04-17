---
title: "索引相关"
linkTitle: "索引相关"
date: 2020-03-07
description: >
  本节讲解和监控索引相关的运维操作
---

## 清理监控索引
有些监控指标在用户不再上报之后，还会在页面保留一段时间(默认是1天)，有的用户会觉得影响看图操作，想把不再上报的指标删掉，所以我们提供了两种删除索引的接口

#### 1.根据endpoint+metric删除索引
```json
//body
{
	"endpoints":["127.0.0.1"],
	"metrics":["cpu.idle"]
}
```
```bash 
curl -X DELETE -d body http://index.addr/api/index/metrics
```
#### 2.根据endpoint+metric+tagkv删除索引
```json 
//body
{
	"endpoints":["127.0.0.1"],
	"metric":"cpu.idle",
	"tagkv":[
		{
			"tagk":"",
			"tagv":[""]
		}
	]
}
```
```bash 
curl -X DELETE -d body http://index.addr/api/index/counter
```

## 重建监控索引
当index重启的时候，内存的索引数据就会丢失，而tsdb模块对发给过index的索引有缓存，短期不会重复发送，虽然index有数据持久化，但在只有一个index的实例的情况下，在进程终止的时候，一些新的索引数据如果没有来得及落盘，就会丢失，导致索引查不到，所以我们提供了重建全量索引的接口，通过调用接口，可以通知tsdb模块，将其内存中的监控指标的索引全量更新一遍

```bash 
curl http://tsdb.addr/api/tsdb/update-index
```
