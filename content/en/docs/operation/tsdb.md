---
title: "绘图相关"
linkTitle: "绘图相关"
date: 2020-03-08
description: >
  本节讲解和绘图相关的运维操作
---

### 绘图功能高可用

默认情况下，用作看图的数据会通过分片的方式，存到单个tsdb实例，如果某个tsdb实例所在的机器宕机，部分曲线看图会受到影响，如果预算充足，可以部署双活集群，配置方式是修改transfer模块的配置文件，每个节点配置两个tsdb实例，样例如下，集群地址之间用逗号分隔

```yaml
backend:
  cluster:
    tsdb01: cluster1.addr:5821,cluster2.addr:5821
```

通过配置双活实例，当有一个实例宕机的时候，查询链路会自动去从另一个实例查询数据，无需手动处理

### 置绘图数据的存储时长

tsdb默认的存储时长配置如下，可以根据自身需求在配置文件中修改，如果监控指标上报周期(step)为10s，则最久可以存储一年。

```yaml
rrd:
  rra:
    1:    720,   # 存储720个原始点
	6:    4320,  # 6点个归档为一个点，保存4320个点
	180:  1440,  # 180个点归档为一个点，保存1440个点
	1080: 2880,  # 1080个点归档为一个点，保存2880个点
```

如果未来看图的时候，希望能看到跟一周之前的数据的对比，即周环比数据，第二条归档策略，建议由4320调整为11520，这样的话归档精度一致，比较容易看图，并且利于未来做环比告警策略，但是，这也带来了弊端，就是监控数据存储的量变大了。

什么情况需要看周环比数据？变化很规律的业务数据，比如订单量、用户在线数，而且还得是体量很大的情况，曲线才会比较平滑，如果只是看机器监控，或者业务体量比较小，意义不大。

### 存储扩容
当tsdb所在的机器资源不足的时候，可以对tsdb模块进行扩容，下面讲一下tsdb如何进行扩容
    
举个例子：   
旧tsdb集群列表为 172.26.19.33，扩容之后tsdb集群列表为 172.26.19.33、172.26.19.34   

1. 首先修改tsdb模块的配置文件，打开migrate扩容开关，将旧的tsdb机器列表写在oldCluster下面，将新的tsdb机器列表写在newCluster下面
```yaml
rrd:
  storage: data/5821
cache:
  keepMinutes: 120
logger:
  dir: logs/tsdb
  level: WARNING
  keepHours: 2
migrate:
  enabled: true
  oldCluster:
    tsdb01: 172.26.19.33:5821
  newCluster:
    tsdb01: 172.26.19.33:5821
    tsdb02: 172.26.19.34:5821
```
2. 配置文件修改完成之后，将旧机器的tsdb模块重启，将新机器的tsdb模块启动
3. 修改transfer配置文件中的backend.cluster字段，将待扩容的tsdb机器添加到cluster下面，然后重启transfer集群
```yaml
backend:
  # in ms
  # connTimeout: 1000
  # callTimeout: 3000
  cluster:
    tsdb01: 172.26.19.33:5821
    tsdb02: 172.26.19.34:5821
```
4. 在监控系统中查看监控指标 n9e.tsdb.migrate.old.out 的变化，当 n9e.tsdb.migrate.old.out 变为0时，表示扩容操作完成，关闭tsdb配置文件中的migrate开关，重启tsdb模块，完成扩容