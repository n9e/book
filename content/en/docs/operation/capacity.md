---
title: "Capacity assessment"
linkTitle: "Capacity assessment"
date: 2020-04-12
description: >
  This section explains the capacity assessment of each module and how to determine whether it needs to be expanded.
---

## Preface
You can determine whether you need to expand capacity in two ways:
1. By viewing the load, cpu, memory, io usage of the machine, and  where the instance is located;
2. View the processing capabilities of the module itself. This section focuses on the business indicators that each module uses to determine whether it has reached its own processing limit.
After each component of Nightingale starts, it will report some of its own status data to the monitoring system, and the reported indicators can determine whether the system needs to be expanded.
The following describes the monitoring indicators of each module:

## transfer
The core monitoring indicators of transfer are as follows, with a statistical period of 10s.

| Monitoring indicators        | meaning   |
| --------   | ----- |
| n9e.transfer.points.in     | Number of receiving points| 
| n9e.transfer.points.out.tsdb        |Number of points sent to tsdb| 

If the number of points sent to tsdb is always much smaller than the number of received points, it means that the transfer capability of transfer has reached a bottleneck and needs to be expanded.
