---
title: "监控策略"
linkTitle: "监控策略"
date: 2020-02-28
description: >
  Nightingale因为内置了服务树这种机器分组机制，和Open-Falcon相比，告警灵活性是一个质的提升
---

Nightingale的告警策略与Open-Falcon的配置有很大区别。首先取消了策略模板的机制，每一条策略都可以单独配置告警接收人，其次，策略可以直接绑定到服务树节点上，节点下的所有机器都会继承生效，另外还增加了一些字段，下面挨个字段解释：

- **策略名称**：描述这条策略的作用，比如“CPU利用率超过85%”
- **生效节点**：关联的服务树节点，节点下所有机器都会应用这条策略
- **排除节点**：生效节点下面的部分子节点可能较为特殊需要排除，可以用此配置解决
- **报警级别**：分三级，P1最严重，报警之后事件通过所有报警通道推送，P3不严重，只用部分通道
- **统计周期**：判断报警的时候使用最近多长时间以内的数据
- **触发条件**：支持与条件，即两个条件都满足才报警
- **Tag过滤**：可以配置只生效监控指标的部分tag，或者排除部分tag，比如disk.io.util只监控sda
- **执行动作**：配置报警收敛策略和报警接收人，也支持配置回调，与自动化逻辑打通
- **留观时长**：告警恢复后持续观察多少秒，称为留观时长，未再触发阈值才发送恢复通知
- **静默恢复**：即只发送告警消息，不发送恢复通知，默认会发送，即不开启静默恢复
- **生效时间**：即策略生效时间，默认7*24生效，可以配置只生效部分时间段


策略配置页面支持导入，这里整理了一些常见策略，可以一键导入，然后批量修改一下报警接收人就可以用起来了 :-)

```json
[
    {
        "name": "timewait状态tcp连接超过2万",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 3,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "net.sockets.tcp.timewait",
                "params": [],
                "threshold": 20000
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "内存利用率大于75%",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "mem.bytes.used.percent",
                "params": [],
                "threshold": 75
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "机器loadavg大于16",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "cpu.loadavg.1",
                "params": [],
                "threshold": 16
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "某磁盘无法正常读写",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 1,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "disk.rw.error",
                "params": [],
                "threshold": 0
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "监控agent失联",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 1,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": "=",
                "func": "nodata",
                "metric": "proc.agent.alive",
                "params": [],
                "threshold": 0
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "磁盘利用率达到85%",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 3,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "disk.bytes.used.percent",
                "params": [],
                "threshold": 85
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "磁盘利用率达到88%",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "disk.bytes.used.percent",
                "params": [],
                "threshold": 88
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "磁盘利用率达到92%",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 1,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "disk.bytes.used.percent",
                "params": [],
                "threshold": 92
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "端口挂了",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": "!=",
                "func": "all",
                "metric": "proc.port.listen",
                "params": [],
                "threshold": 1
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "网卡入方向丢包",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "net.in.dropped",
                "params": [],
                "threshold": 3
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "网卡入方向错包",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "net.in.errs",
                "params": [],
                "threshold": 3
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "网卡出方向丢包",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "net.out.dropped",
                "params": [],
                "threshold": 3
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "网卡出方向错包",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "net.out.errs",
                "params": [],
                "threshold": 3
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "进程总数超过3000",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 1,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": ">",
                "func": "all",
                "metric": "sys.ps.process.total",
                "params": [],
                "threshold": 3000
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    },
    {
        "name": "进程挂了",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
        "runbook": "",
        "nids": null,
        "exprs": [
            {
                "eopt": "<",
                "func": "all",
                "metric": "proc.num",
                "params": [],
                "threshold": 1
            }
        ],
        "tags": [],
        "enable_days_of_week": [
            0,
            1,
            2,
            3,
            4,
            5,
            6
        ],
        "converge": [
            36000,
            1
        ],
        "endpoints": null,
        "judge_instance": "",
        "work_groups": null
    }
]
```

