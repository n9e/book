---
title: "Process monitoring"
linkTitle: "Process monitoring "
date: 2020-02-23
description: >
  Process monitoring in Nightingale by configuring specific collection strategies to specify which processes to collect, and also supports reading the target machine identification file
---


Two configuration methods are supported, one is to create a meta information file in the specified directory of the target machine, and the other is to configure the collection strategy on the page.
## Target machine meta information file method

The processing logic of process monitoring and port monitoring is similar. Both touch a file locally and tell the collector which process to monitor. If you want to monitor multiple processes, you need to touch multiple files.

For port monitoring, the port number is unique, but for processes, the process name (see process name can be `cat /proc /<pid> /status`) is not unique, such as processes written in Python The names are all processes written in python and java, and the process names are java. Then what kind of file should we touch to tell the collector exactly which process we want to collect?

We need to find something that can easily distinguish a process, here we have two ways. One is to use the process name, and the other is to use the process cmdline (see `/proc /<pid> /cmdline`),For example, if you touch a 10_name_n9e-monapi file, you can collect a process named n9e-monapi. Because n9e-monapi is written by go, the process name is n9e-monapi, so this configuration is OK.If the process name cannot be distinguished well, you can touch a 10_cmdline_xx file, assuming that the process pid to be monitored is 5648, and xx is a substring of `/proc /5648 /cmdline`.As long as this substring can be distinguished from the cmdline of other processes, there is no need to use the entire cmdline content.Example: We started a java process using `java -c uic.properties`, and its cmdline is:` java-cuic.properties`. To monitor this process, we only need to touch a 10_cmdline_uic.properties file.

For the file content, just like the port monitoring, you can write the module name of the process. When the monitoring data is reported, it will bring this information and report it as a tag
## Configure the collection strategy on the page

This method is relatively simple, but I will repeat it. There is a service field during the configuration of the collection strategy. You need to fill in the name of the module, which is equivalent to the content part of the target machine meta information file.