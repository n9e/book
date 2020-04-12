---
title: "Change check"
linkTitle: "Change check"
date: 2020-04-12
description: >
  This section explains the change checklist of each module
---

## Preface
There are two ways to determine whether the change is abnormal:
1. Determine whether the process after the change is still alive. This method can be achieved by adding process monitoring to the module;
2. By observing whether the indicators of the module are normal. This section focuses on the monitoring indicators that each module needs to pay attention to.
    
After each component of Nightingale is started, it will report some of its own status data to the monitoring system. You can observe whether the system is operating normally through the reported indicators. The monitoring indicators of each module are introduced below.
## transfer change
The core indicators of `transfer` are as follows, with a statistical period of 10s.

| Monitoring indicators       | meaning  |
| --------   | ----- |
| n9e.transfer.points.in     |  Number of receiving points| 
| n9e.transfer.points.out.tsdb        |Number of points sent to tsdb| 
| n9e.transfer.points.out.tsdb.err        |Number of failed points sent to tsdb| 
| n9e.transfer.points.out.judge        |Number of points sent to judge| 
| n9e.transfer.points.out.judge.err        |Number of failed points sent to judge| 
| n9e.transfer.stra.count        |Get the number of monitoring strategies| 

If there is no obvious change in the above indexes of the transfer cluster after the change of transfer, then the change is in line with expectations.

## tsdb change
The core indicators of `tsdb` are as follows, with a statistical period of 10s

| Monitoring indicators        | meaning   |
| --------   | ----- |
| n9e.tsdb.points.in     | Number of receiving points| 
| n9e.tsdb.query.miss     | Number of times the query data is empty| 
| n9e.tsdb.index.out.err     | Number of failed index pushes| 

If there is no obvious change in the above indicators of the tsdb cluster after the change of tsdb, then the change is in line with expectations.

## index change
The core indicators of `index` are as follows, with a statistical period of 10s

| Monitoring indicators       | meaning   |
| --------   | ----- |
| n9e.index.query.counter.miss     |Number of misses when querying index with fullmatch| 
| n9e.index.xclude.miss     |xclude query index misses| 

If there is no obvious change in the index data of the index cluster after the index is changed, the change is in line with expectations.

## judge change
The core indicators of `judge` are as follows, the statistical period is 10s

| Monitoring indicators        | meaning   |
| --------   | ----- |
| n9e.judge.push.in     |Number of receiving points| 
| n9e.judge.running     |Number of judge tasks being executed| 
| n9e.judge.stra.count     |Number of strategies acquired| 

If there is no obvious change in the above indicators of the judge cluster after the change of judge, it means that the change is in line with expectations.
     