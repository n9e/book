---
title: "nodata监控"
linkTitle: "nodata"
date: 2021-05-15
description: >
  使用prometheus的absent函数。采用那些没有tags的指标，如果指标有tags，需要所有的tags都没有数据才会触发
---

#### 配置说明

- 配置告警策略时选择prometheus类型
- 使用absent函数
- `==1`代表函数内部的series不存在，区分具体的tag
- 具体如下 nodata_test_metric_not_exist是没有监控数据的metrics，`==1`就可以告警

```shell script
absent(nodata_test_metric_not_exist)==1
```

*真实生产中应该配置一个具有 up 指标含义的 metrics 来做 nodata 告警*
