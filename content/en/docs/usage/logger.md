---
title: "Log monitoring"
linkTitle: "Log monitoring"
date: 2020-02-21
description: >
  Compared with Open-Falcon, Nightingale has built-in log monitoring. You can place the log collection strategy file on the target machine, or you can configure the log collection strategy on the page
---

Log monitoring is an external collection. By reading the log printed by the process, the monitoring data is collected and aggregated. After being aggregated into standard time series data, it is pushed to Nightingale's unified back-end storage.

Log monitoring supports two configuration methods, one is to create a meta information configuration file in the specified directory of the target machine; one is to configure the collection strategy on the page. The original code of the log collection part is forked from：[falcon-log-agent](https://github.com/didi/falcon-log-agent)。

## Meta information configuration file method

Check the collector configuration file, logPath is the configuration for this item (default value: /home /n9e /etc /log）。You can put the log collection strategy file (name format `log. <Xx> .json`) into this directory. The collector component will automatically detect and collect.
for example:

```json
{
    "name": "log.sys.oom",
    "file_path": "/var/log/messages",
    "time_format": "mmm dd HH:MM:SS",
    "pattern": "Out of memory",
    "interval": 10,
    "tags": {},
    "func": "cnt",
    "degree": 6,
    "unit": "frequency",
    "comment": "Process oom"
}
```

- **name**：Indicator name, corresponding to the metric field of the data after reporting
- **file_path**：Log files to be collected
- **time_format**：Timestamp format used when the log is printed
- **pattern**：Regular expression, the system will match the log file specified by file_path line by line according to the configured regularity
- **interval**：Acquisition cycle, that is, how many seconds to generate a monitoring data point
- **tags**：Support to collect other dimensions needed by regular way and use different tags to mark
- **func**：Calculation method, that is, the method of calculating the collected multi-line logs to obtain the last time point
- **degree**：Calculation accuracy, the accuracy adopted when the value of the monitoring index is finally calculated, just keep the default
- **unit**：The unit of the monitoring indicator is only used as a marker and does not change the monitoring value
- **comment**：Remarks, descriptive information

What does the above json configuration mean? It means that the /var /log /messages file will be collected every 10s, matching the keyword "Out of Memory", how many lines can be matched (ie cnt function), and the number of lines is reported as the value of the monitoring index. In this way, the server can configure an alarm strategy. When the value of log.sys.oom is greater than 0, it will alarm, indicating that OOM appears in the system.

Below we explain the key fields in more detail:

**file_path**

The file path supports both fixed and dynamic paths. The name of the log file of the fixed path will not change, the purpose of the dynamic path is to support the log file that is cut by time slice and reflected in the file name。
For example, the above /var /log /messages is a fixed path; examples of dynamic path formats： `/opt/appx/log/${%Y%m%d}/appx.log.${%Y%m%d%H}`

**time_format**

Log monitoring requires a certain time stamp in the log to ensure the accuracy of the calculation. In the case of delayed log placement and a large log volume, as long as the calculation bottleneck is not reached, the accuracy of the data can still be guaranteed.

The time formats currently supported by Nightingale are as follows:

```
dd/mmm/yyyy:HH:MM:SS
dd/mmm/yyyy HH:MM:SS
yyyy-mm-ddTHH:MM:SS
dd-mmm-yyyy HH:MM:SS
yyyy-mm-dd HH:MM:SS
yyyy/mm/dd HH:MM:SS
yyyymmdd HH:MM:SS
mmm dd HH:MM:SS
```

**pattern**

The regular expression used to match. The regular expression is used to match the lines in the log that meet the conditions, and the hit lines are summarized and calculated.

Regular expressions in Golang use RE2 syntax, and RE2 removes backreferences and zero-width assertions. Therefore, if we want to exclude some keywords, we can use our built-in syntactic sugar:

```
```EXCLUDE```
```

You can use this syntactic sugar to split the front hit regular configuration and the rear exclude regular configuration and assemble it into a pattern.

E.g:

```
code=[45]00```EXCLUDE```SpeciallyErrorNo
```

**func**

Collection method **func **, when a batch of rows that meet the rules is filtered out through the configured regularity, in what way should be summarized, and then we can get the results we want.

The currently supported summary methods are:

- cnt
- avg
- sum
- max
- min

```
Examples are as follows:
Assume that the regular expression is configured as Return Success: (\ d +) s Used
 
The log rolls in a certain period:
2017/12/01 12:12:01 Return Success : 1s Used
2017/12/01 12:12:02 Return Success : 2s Used
2017/12/01 12:12:03 Return Success : 4s Used
2017/12/01 12:12:04 Return Success : 2s Used
2017/12/01 12:12:05 Return Success : 1s Used
 
First, get the values ​​in parentheses according to the regularity: 1, 2, 4, 2, 1
Next, according to different calculation methods, you will get different results:
avg   : (1 + 2 + 4 + 2 + 1) / 5 = 2
cnt   : 5
sum   : (1 + 2 + 4 + 2 + 1) = 10
max   : Max(1, 2, 4, 2, 1) = 4
min   : Min(1, 2, 4, 2, 1) = 1
```

**tags**

The log in the func section is relatively simple, just a time-consuming print. If our system has multiple interfaces, we hope to collect the interface information corresponding to the time-consuming at the same time, and report it as tags, so that the server sees In the picture, you can clearly know the time-consuming situation of each interface. What should you do?

```
For example, the log rolls in a certain period:
2017/12/01 12:12:01 Return Success : 1s Used by /a
2017/12/01 12:12:02 Return Success : 2s Used by /b
2017/12/01 12:12:03 Return Success : 4s Used by /a
2017/12/01 12:12:04 Return Success : 2s Used by /a
2017/12/01 12:12:05 Return Success : 1s Used by /b

When our pattern does not change, still: Return Success: (\ d +) s Used, pattern just means whether it can match a certain line of logs

The tags field is now configured as:
{
    "...": "...",
    "tags": {
        "api": "Used by (\\/\\w+)"
    },
    "...": "..."
}

Just to extract strings like /a or /b, in fact, the regular configuration can be Used by (\ /\ w +), but,
This regular is configured in the "" of json, \ needs to be escaped, so it becomes the above, if it is configured on the web side, no escape is required
```

Note: **If the tag value cannot be matched, it is deemed that the data is not matched, and the log will no longer be counted. **

## Configure the collection strategy on the page

The fields of the page function basically correspond to the configuration files one by one, but I won't repeat them here. In addition, there is a field for configuration verification on the web, you can put a real log to see if the regular can be properly matched.
