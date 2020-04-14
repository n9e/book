---
title: "Alarm related"
linkTitle: "Alarm related"
weight: 2
date: 2020-04-14
description: >
  This section explains common problems related to alarms
---

### Q1:The alarm strategy has been configured, and the monitoring indicators are abnormal but no alarm has been sended?

May be the following situations:

1. **View object mount**：The object (usually a machine) needs to be mounted under the node (or its descendants) that configures the policy
1. **View monitoring data**：Confirm that the monitoring data corresponding to the monitoring strategy is valuable and will definitely trigger the threshold
1. **Incorrect threshold setting**：Check the alarm function settings of the alarm strategy, confirm whether the alarm trigger condition is met, and confirm whether the effective time of the strategy is met
1. **Policy is blocked**：Check the alarm policy blocking list to confirm whether the policy is blocked
1. **Notify gateway problems**：On the alarm history page, check whether there is an alarm event that has been generated. If there is an alarm event, the problem is to notify the gateway
1. **Issue of strategy**：
	- Run `curl '127.0.0.1: 5800 /api /portal /stras /effective? All = 1'` to get a list of all strategies
	- If the list does not have this strategy, check the monapi logs to see if there are any error messages. If there are errors, follow the prompts
	- If so, check the judge_instance field, find out which instance of the judge the strategy is distributed to, and log in to the machine where the judge is located,
	Run `curl 127.0.0.1: 5840 /api /judge /stra /: id` to see if it has been sent to judge
1. **judge解析策略异常**：Check the judge WARNING.log and ERROR.log logs to check if there is any error message for this strategy
1. **judge没有收到数据**：Modify the judge log level to DEBUG, `tail -f DEBUG.log | grep monitoring indicator` No log output, indicating that the data has not reached the step of judge
1. **查看数据链路**：Check the transfer log and analyze why the data is not transferred from the transfer to the judge module

