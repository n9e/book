---
title: "Plugin mechanism "
linkTitle: "Plugin mechanism"
date: 2020-02-19
description: >
  Nightingale can designate a certain directory as a plug-in directory, read the files in this directory that conform to a specific format, and execute it as a plug-in, thereby expanding the collector's ability
---

Although the collector component has a built-in collection of monitoring indicators, it cannot solve all scenarios, so a plug-in mechanism is provided to extend the function of the collector。

Check the collector configuration file, the plugin directory is configured inside, we only need to put the plugin script in this directory, the collector will automatically detect it, and then run periodically. For the collection of monitoring indicators of the business system, you can put the collection script in the business program release package and go online as the business code goes online (put the script in the collector's plugin directory when going online), and upgrade as the business code upgrades. As the business code goes offline and goes offline (remove the script from the collector's plugin directory when going offline), it will be easier to manage.

For plugins, there are several requirements:

- The plugin script must have executable permissions, remember to execute chmod + x after the script is deployed
- The plugin script can be sh, py, pl, rb, or even binary, as long as there is a runtime environment on the machine
- Plugin script naming: `$ {step} _xx.xx`, such as` 20_uptime.sh`, `$ {step}` is telling the collector how often to run
- Files or directories in the plugin directory that are not in the `$ {step} _xx.xx` naming format can exist, it does not matter, the collector will not recognize it as a plugin
- After the plugin is executed, a json array must be output on stdout, and the collector will intercept this output and parse it as a monitoring indicator for reporting
- If the plug-in executes an error, the error message should be printed to stderr, not stdout

The following is an example of a plugin written by shell 20_uptime.sh:

```bash
#!/bin/bash

duration=$(cat /proc/uptime | awk '{print $1}')
localip=$(ifconfig `route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|awk -F ':' '{print $NF}'|head -n 1)
step=$(basename $0|awk -F'_' '{print $1}')
echo '[
    {
        "endpoint": "'${localip}'",
        "tags": "",
        "timestamp": '$(date +%s)',
        "metric": "sys.uptime.duration",
        "value": '${duration}',
        "step": '${step}'
    }
]'
```

The following is an example of a plugin written in python 20_plugin_status.py:

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import time
import commands
import json
import sys
import os

items = []


def collect_myself_status():
    item = {}
    item["metric"] = "plugin.myself.status"
    item["value"] = 1
    item["tags"] = ""
    items.append(item)


def main():
    code, endpoint = commands.getstatusoutput(
        "timeout 1 ifconfig `route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|awk -F ':' '{print $NF}'|head -n 1")
    if code != 0:
        sys.stderr.write('cannot get local ip')
        return

    timestamp = int("%d" % time.time())
    plugin_name = os.path.basename(sys.argv[0])
    step = int(plugin_name.split("_", 1)[0])

    collect_myself_status()

    for item in items:
        item["endpoint"] = endpoint
        item["timestamp"] = timestamp
        item["step"] = step

    print json.dumps(items)


if __name__ == "__main__":
    main()
```

In addition, the Open-Falcon community can see that many third-party indicator collectors do not exist as plug-in mechanisms. They are all in the form of independent daemon processes or cron scripts. This method is similar to plug-ins, except that you need to monitor the collection yourself Pushing data to the collector or transfer is also a kind of generalized plug-in.