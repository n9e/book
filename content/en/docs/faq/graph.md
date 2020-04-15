---
title: "Drawing related"
linkTitle: "Drawing related"
weight: 1
date: 2020-04-14
description: >
  This section explains common problems related to acquisition and monitoring charts
---

#### Q1:The monitoring indicators have been sent to the server, but in the end the graph cannot be seen

In this case, the data format is generally incorrect, or the data is not reported to the server.Suppose the sent index is n9e.points.in
1. First look at the transfer and tsdb modules for n9e.points.in error logs 
Log in to the machine where the transfer is deployed and execute `tail -f /home/n9e/logs/transfer/WARNING.log|grep n9e.points.in`   
登Lu to the machine where tsdb is deployed, execute `tail -f /home/n9e/logs/tsdb/WARNING.log|grep n9e.points.in`
2. If there is no error message, confirm again whether the monitoring index is reported to the server, and modify the transfer log level to DEBUG   
excute `tail -f /home/n9e/logs/transfer/DEBUG.log|grep n9e.points.in` If no log appears, it means that it has not been reported to the server, and troubleshoot the sender

#### Q2:The monitoring plugin is placed in the specified directory, but the picture cannot be seen in the end

1. Encounter this kind of problem is generally because the plugin execution failed.For example, the plug-in does not have executable permissions, or the data format reported by the plug-in is incorrect.You can log in to the machine in question and check /home/n9e/logs/collector/ERROR.log,Check whether there are error logs related to the plug-in, and if so, resolve them. If there is no error log, follow [Q1] (# q1 monitoring indicators have been sent to the server but the picture cannot be seen eventually)

#### Q3:Write a program and report the data to the collector or transfer according to [Data Specification] (../usage/metric/), but you cannot see the picture

1. First, print out the request body and resp body of the program. Check the err in the resp. If it is not empty, there is a problem with the report format. If the request body is empty, there is no data reported.
2.If the report is normal, check according to the idea of ​​[Q1] (# q1 monitoring indicators have been sent to the server but the picture cannot be seen eventually)

#### Q4:Log, process or port collection is configured, but the relevant monitoring chart cannot be seen

1. Check whether the collection strategy has been delivered to the collector of the target machine       
`curl IP of target machine:2058/api/collector/stra`
2. If you can check the configured collection result, run tail -f /home/n9e/logs/collector/ERROR.log to check whether there is an error log
3. If there is an error, follow the log prompt to deal with it. If no error is reported, follow [Q1] (# q1 monitoring indicators have been sent to the server but the picture cannot be seen in the end).