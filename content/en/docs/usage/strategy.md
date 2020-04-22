---
title: "Monitoring strategy "
linkTitle: "Monitoring strategy "
date: 2020-04-22
description: >
  Because Nightingale has a built-in machine grouping mechanism such as service tree, compared with Open-Falcon, the alert flexibility is a great improvement
---


The monitoring strategy is very different from the configuration of Open-Falcon. First of all, the mechanism of the policy template is canceled. Each policy can be configured with an alarm recipient separately. Secondly, the policy can be directly bound to the service tree node. All machines under the node will inherit and take effect. In addition, some fields have been added. Field explanation:

- **Strategy name **: Describe the role of this strategy, such as "CPU utilization exceeds 85%"
- **Effective node **: The associated service tree node, all machines under the node will apply this strategy
- **Exclude node **: Some of the sub-nodes below the effective node may need to be excluded due to special circumstances, which can be solved with this configuration
- **Alarm level **: divided into three levels, P1 is the most serious, after the alarm, the event is pushed through all alarm channels, P3 is not serious, only some channels are used
- **Statistical Period **: Use the data within the most recent time to judge the alarm
- **Trigger condition **: Support and condition, that is, only two conditions are met before the alarm
-**Tag filtering **: You can configure only some tags for monitoring indicators to take effect, or exclude some tags, such as disk.io.util only monitor sda
-**Execution Action **: Configure alarm convergence strategy and alarm recipient, and also support configuration callback to communicate with automation logic
-**Watch time **: How many seconds to keep watching after the alarm is restored, which is called watch time, and the recovery notification is sent without triggering the threshold again
-**Silent Recovery **: That is, only the alarm message is sent, and the recovery notification is not sent. By default, it will be sent, that is, the silent recovery is not turned on
-**Effective time **: the effective time of the policy, the default is 7 *24 effective, you can configure only effective part of the time period

The policy configuration page supports importing. Here are some common policies, which can be imported with one click, and then the alarm recipient can be modified in batches :-)

```json
[
    {
        "name": "Memory utilization is greater than 75%",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
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
        "endpoints": null
    },
    {
        "name": "Machine loadavg is greater than 16 ",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
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
        "endpoints": null
    },
    {
        "name": "A disk cannot be read and written normally ",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 1,
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
        "endpoints": null
    },
    {
        "name": "Monitoring agent lost contact ",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 1,
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
        "endpoints": null
    },
    {
        "name": "Disk utilization reaches 85% ",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 3,
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
        "endpoints": null
    },
    {
        "name": "Disk utilization reaches 88% ",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
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
        "endpoints": null
    },
    {
        "name": "Disk utilization reaches 92% ",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 1,
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
        "endpoints": null
    },
    {
        "name": "Port hung ",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
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
        "endpoints": null
    },
    {
        "name": "The network card loses packets in the incoming direction",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
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
        "endpoints": null
    },
    {
        "name": "The network card loses packets in the outbound direction",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
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
        "endpoints": null
    },
    {
        "name": "The total number of processes exceeds 3000",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 1,
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
        "endpoints": null
    },
    {
        "name": "Process hung",
        "category": 1,
        "alert_dur": 60,
        "recovery_dur": 0,
        "recovery_notify": 1,
        "enable_stime": "00:00",
        "enable_etime": "23:59",
        "priority": 2,
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
        "endpoints": null
    }
]
```
