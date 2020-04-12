---
title: "Self-monitoring"
linkTitle: "Self-monitoring"
date: 2020-04-12
description: >
  This section explains how to do self-monitoring of the monitoring system
---

To do a good job of self-monitoring of the monitoring system, we start from three aspects: preventing problems, finding problems in a timely manner, and quickly positioning stop loss.

### Prevent problems
Through the establishment of monitoring dashboard, regularly observe the change trend of core monitoring indicators to discover potential problems in advance. Each module of Nightingale will actively report some of its own monitoring data, through which we can build a monitoring dashboard and conduct regular inspections.Related monitoring indicators are mentioned in [Change Check] (../checklist/).

### Timely detection of problems

A well-configured alarm strategy is the main means of finding problems in a timely manner.The strategy configuration page supports import operations. We have compiled some self-monitored [alarm strategies] (https://s3-gz01.didistatic.com/n9e-pub/json/stra.json), you can import these indicators, and then can be used by modifying the alarm recipient. :-) 


However, when the data forwarding, alarm, and notification components of the monitoring system are unavailable, their own alarm capabilities will be unavailable, so a third-party service is also required to monitor the module survival。When the alarm capability of the monitoring system fails, we can still receive the alarm notification。Third-party monitoring should be simple enough. The self-monitoring service used by open-falcon is recommended here [anteye] (https://github.com/niean/anteye)，Add a process survival monitoring alarm to anteye, so that the two monitoring systems monitor each other, and if any party has a problem, we can timely sense.

### Quickly locate problems
After receiving the system's own alarm, if you can quickly locate the problem from the alarm notification, you can directly perform a stop loss operation.
If you cannot locate the problem immediately, it is recommended to first observe the inspection dashboard created in advance to understand the overall status of the service, and then observe the system and business indicators of the problem module to gradually converge the problem and quickly locate the stop loss.