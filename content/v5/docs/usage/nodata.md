---
title: "nodata监控"
linkTitle: "nodata监控"
date: 2021-08-02
description: >
    使用prometheus的absent函数
---

# 配置说明
- 配置告警策略时选择prometheus类型
- 使用absent函数
- ==1代表函数内部的series不存在，区分具体的tag
- 具体如下 nodata_test_metric_not_exist是不在的metrics，==1就可以告警
```shell script
absent(nodata_test_metric_not_exist)==1
```
- 真实生产中应该配置一个具有up指标含义的metrics作为 nodata告警
