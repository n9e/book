---
title: "Port monitoring"
linkTitle: "Port monitoring"
date: 2020-02-23
description: >
  Port monitoring In Nightingale, you can specify which ports to collect by configuring specific collection strategies, and also support reading the target machine identification file
---


Two configuration methods are supported, one is to create a meta information file in the specified directory of the target machine, and the other is to configure the collection strategy on the page.

## Target machine meta information file method

You can view the configuration file of the collector. There is a portPath configuration item. You can put a file of the port to be monitored on the path specified by portPath. The collector component will automatically detect and automatically collect.

For example, to collect the survivability of port 5800 of the monapi component of the n9e service, you can create a 10_5800 file under the path specified by portPath (if not, manually create the corresponding directory), the content is n9e-monapi.

- File name naming convention `$ {acquisition frequency} _ $ {acquisition port}`
- The content of the file is the name of this module, such as n9e-monapi. When we call the alarm, we see n9e-monapi and we know which service has a problem. Special characters such as spaces and quotation marks are not allowed.

For business services, when we deploy services, we can create this port file, so that the platform can help us monitor this service. When we go offline, we delete this port file, so the platform This port is no longer monitored.

After the port collection is completed, we can configure a monitoring strategy on the public parent node of the service in charge, and we can get all ports of all modules of all services monitored:

> The collection period is assumed to be 20 seconds, and the policy statistics period can be configured to 60 seconds. Each value = 0 is an alarm. The metric for port monitoring is: proc.port.listen. There is no need to configure each port separately.

## Configure the collection strategy on the page

This method is relatively simple, but it will be described in more detail. There is a service field when configuring the collection strategy. You need to fill in the name of the module, which is equivalent to the content part of the target machine meta information file.

