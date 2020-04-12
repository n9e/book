---
title: "Drawing related"
linkTitle: "Drawing related"
date: 2020-04-12
description: >
  This section explains operations related to drawing
---

### Highly available of drawing

By default, the data used to view the image will be saved to a single tsdb instance through sharding.If the machine where a tsdb instance is down, some graphs will be affected.If the budget is sufficient, you can deploy an active-active cluster.By modifying the configuration file of the transfer module, two tsdb instances can be configured for each node. In the example below, the cluster addresses are separated by commas.

```yaml
backend:
  cluster:
    tsdb01: cluster1.addr:5821,cluster2.addr:5821
```

After configuring the HyperMetro instance, when one instance goes down, the query link will automatically go to query data from another instance without manual processing.

### Set the storage time of drawing data

The following is the default storage duration configuration of tsdb, you can modify the configuration file according to your own needs. If the monitoring index reporting step (step) is 10s, it can be stored for up to one year.

```yaml
rrd:
  rra:
    1:    720,   # Store 720 original points
	6:    4320,  # 6 points are archived as one point, and 4320 points are saved
	180:  1440,  # 180 points are archived as a point, saving 1440 points
	1080: 2880,  # 1080 points are archived as one point, and 2880 points are saved
```

If you want to see the comparison chart with the data from a week ago, that is, the week-on-week data, it is recommended to adjust the second archiving strategy from 4320 to 11520. In this case, the archiving accuracy is consistent, it is easier to see the picture, and it is conducive to the future of the ring alarm strategy However, this also brings a disadvantage-the amount of monitoring data storage becomes larger.

When should we look at the week-on-week data? Business data that changes very regularly, such as the number of orders and the number of users online.And it must be a case where the amount of data is large, the curve will be relatively smooth, if you only look at machine monitoring, or the amount of business is relatively small, it does not make much sense.

### Storage expansion
When the machine resource where tsdb is located is insufficient, the tsdb module can be expanded. Let's talk about how to expand tsdb
    
For example:   
The old tsdb cluster list is 172.26.19.33, and the expanded tsdb cluster list is 172.26.19.33, 172.26.19.34

1. First modify the configuration file of the tsdb module. Open the migrate expansion switch, write the old tsdb machine list under oldCluster, and write the new tsdb machine list under newCluster
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
2. After the configuration file is modified, restart the tsdb module of the old machine, and then start the tsdb module of the new machine
3. Modify the backend.cluster field in the transfer configuration file, add the old tsdb machine to the cluster, and then restart the transfer cluster
```yaml
backend:
  # in ms
  # connTimeout: 1000
  # callTimeout: 3000
  cluster:
    tsdb01: 172.26.19.33:5821
    tsdb02: 172.26.19.34:5821
```
4. Observe the change of n9e.tsdb.migrate.old.out in the monitoring system. When n9e.tsdb.migrate.old.out becomes 0, it means that the expansion operation is completed, we can turn off the migrate switch in the tsdb configuration file and restart tsdb, then module expansion is completed.