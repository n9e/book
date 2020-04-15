---
title: "日志监控"
linkTitle: "日志监控"
date: 2020-02-21
description: >
  夜莺相比Open-Falcon，直接内置了日志监控，可以在目标机器放置日志采集策略文件，也可以在页面上配置日志采集策略
---

日志监控，是一种外挂式的采集。通过读取进程打印的日志，来进行监控数据的采集与汇聚计算。汇聚成标准的时间序列数据之后，推送给Nightingale统一的后端存储。

日志监控支持两种配置方式，一种是在目标机器的指定目录创建元信息配置文件，一种是在页面上配置采集策略。日志采集部分最原始的代码是fork自：[falcon-log-agent](https://github.com/didi/falcon-log-agent)。

## 元信息配置文件方式

查看collector的配置文件，logPath就是针对此项的配置（默认取值：/home/n9e/etc/log），可以把日志采集策略文件（命名格式`log.<xx>.json`）放到这个目录，collector组件会自动探测，进行采集。
举个例子：

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
    "unit": "次数",
    "comment": "有进程oom了"
}
```

- **name**：指标名，对应上报后数据的metric字段
- **file_path**：要采集的日志文件
- **time_format**：日志打印时采用的时间戳格式
- **pattern**：正则表达式，系统会根据所配置正则逐行匹配file_path指定的日志文件
- **interval**：采集周期，即每隔多少秒产生一个监控数据点
- **tags**：支持通过正则的方式，采集出所需要的其他维度，使用不同的标签来标记
- **func**：计算方法，即对采集出的多行日志进行计算，得出最后时间点的方法
- **degree**：计算精度，监控指标的值最终计算的时候采用的精度，维持默认即可
- **unit**：监控指标的单位，仅用作标记，并不会改变监控数值
- **comment**：备注，描述信息

如此，上面的这条json配置代表什么意思呢？表示会每隔10s采集一次 /var/log/messages 文件，匹配 "Out of Memory" 这个关键字，看总共可以匹配到多少行（即cnt函数），把行数作为监控指标的值上报。这样服务端就可以配置告警策略，当log.sys.oom的值大于0就报警，说明系统出现了OOM。

下面我们对关键的字段进行更详细的解释：

**file_path**

文件路径支持固定路径和动态路径两种。固定路径的日志文件的名称不会变更，动态路径目的是为了支持按时间片切割并在文件名中有所体现的日志文件。
例如，上面的/var/log/messages就是固定路径；动态路径格式举例： `/opt/appx/log/${%Y%m%d}/appx.log.${%Y%m%d%H}`

**time_format**

日志监控要求日志中必须有确定的时间戳，以保证计算的准确性。在日志落盘延迟、日志量太大的情况下，只要不达到计算的瓶颈，仍然可以保证数据的准确性。

目前Nightingale支持的时间格式如下：

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

用来匹配的正则表达式，正则表达式用来匹配日志中满足条件的行，将命中的行进行归纳和计算。

由于Golang正则表达式使用RE2语法，而RE2去掉了反向引用和零宽断言。因此，如果我们想排除掉一些关键字，可以使用我们内置的语法糖： 

```
```EXCLUDE```
```

可以使用这个语法糖来分割前方的命中正则配置和后方的排除正则配置，组装成一个pattern。

例如：

```
code=[45]00```EXCLUDE```SpeciallyErrorNo
```

**func**

采集方式**func**，当通过配置的正则筛选出一批符合规则的行，应该以何种方式进行归纳汇总，进而得出我们想要的结果。

目前支持的汇总方式有：

- cnt
- avg
- sum
- max
- min

```
举例如下：
假设正则表达式配置为 Return Success : (\d+)s Used
 
某一个周期内日志滚动：
2017/12/01 12:12:01 Return Success : 1s Used
2017/12/01 12:12:02 Return Success : 2s Used
2017/12/01 12:12:03 Return Success : 4s Used
2017/12/01 12:12:04 Return Success : 2s Used
2017/12/01 12:12:05 Return Success : 1s Used
 
首先，根据正则获取到括号内的值：1、2、4、2、1
接下来，根据不同的计算方式，会得到不同的结果：
avg   : (1 + 2 + 4 + 2 + 1) / 5 = 2
cnt   : 5
sum   : (1 + 2 + 4 + 2 + 1) = 10
max   : Max(1, 2, 4, 2, 1) = 4
min   : Min(1, 2, 4, 2, 1) = 1
```

**tags**

func一节举例的日志相对简单，只是一个耗时打印，如果我们的系统有多个接口，我们在采集的时候希望同时采集到耗时对应的接口信息，并将其作为tags上报，这样服务端看图的时候就可以清晰的得知各个接口的耗时情况，应该如何做呢？

```
举例某一个周期内日志滚动：
2017/12/01 12:12:01 Return Success : 1s Used by /a
2017/12/01 12:12:02 Return Success : 2s Used by /b
2017/12/01 12:12:03 Return Success : 4s Used by /a
2017/12/01 12:12:04 Return Success : 2s Used by /a
2017/12/01 12:12:05 Return Success : 1s Used by /b

此时我们的pattern不变，还是：Return Success : (\d+)s Used，pattern只是表示能否匹配到某行日志

tags字段此时配置为：
{
    "...": "...",
    "tags": {
        "api": "Used by (\\/\\w+)"
    },
    "...": "..."
}

只是为了提取 /a 或者 /b 这样的字符串，其实正则配置为 Used by (\/\w+) 即可，但是，
这个正则是配置在json的""内的，\ 需要被转义，所以变成上面这样了，如果是在web端配置，无需转义
```

注意：**如果无法匹配出tag的值，则视为该条数据未匹配到，该条日志将不再计入统计。**

## 页面上配置采集策略

页面功能各字段与配置文件基本一一对应，此处不过多赘述。另外，web上有一个配置验证的字段，可以放一条真实日志看正则是否能够正常匹配。
