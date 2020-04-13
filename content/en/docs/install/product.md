
---
title: "Distributed installation in production environment"
linkTitle: "Production installation"
date: 2020-04-13
description: >
  This section mainly introduces how to install distributed modules, which is suitable for formal production environment. Service stability is very important, so a monitoring system that can detect problems in time is very important. It is necessary to use several more machines to improve disaster tolerance.
---


For disaster recovery considerations, the deployment of Nightingale in the production environment generally uses at least 2 machines (mysql and redis are separately deployed for high availability, and they are not mixed on these two machines). If you want to monitor more devices and services, you need to add more machines. For a scenario of about 1000 devices, it is best to have at least 2 models with 16C /32G /300G SSD or more.

In section《[Source code compilation and installation](../compile/)》，We have already introduced how to get the binary distribution package and how to import the database. This section will not go into details; it is assumed that mysql and redis have been built; this section mainly explains the correlation configuration part of each module.

### Architecture review

Here again show the architecture diagram for reference. Knowing the architecture and building the system will do more with less. We will introduce the general functions and configuration points of each module.
![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-arch.png)

### Write in front

Since it is a production environment, all modules must be able to be automatically pulled up when they are hung up, and must be set to start automatically after booting.You can use hosting such as systemd or supervisor, and you can find examples of service files for each module in the etc /service directory.In addition, all modules that depend on mysql read mysql.yml uniformly, so the connection configuration of mysql will only appear in mysql.yml.

### Association configuration

Each module has its own listening port, which is related to other modules and requires various associated access. We uniformly extract a configuration file of address.yml, configure the address of each module listen, and the list of machines deployed by each module for the caller to use.Therefore, when all modules are deployed separately, not only the configuration file with the same name but also address.yml

[A small video explaining how to use address.yml](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-usage-address.mp4)

### monapi

`monapi` mainly accepts web-side requests. It is stateless and can be deployed horizontally to deploy multiple units. It depends on mysql and redis. Please prepare the password in advance.There are three configuration files that `monapi` depends on. In addition to address.yml, the other is mysql.yml and monapi.yml. The database connection configuration in mysql.yml, it is relatively simple, here mainly talk about monapi.yml.

```yaml
---
# This is a random string used for password encryption, given a random string
salt: "PLACE_SALT"

logger:
  dir: "logs/portal"
  level: "WARNING"
  keepHours: 24

# secret is a random string of encrypted cookies, just give a random string
http:
  secret: "PLACE_SECRET"

# LDAP authentication related configuration, please coordinate with the LDAP administrator to help with configuration, they know this knowledge best
# If you are not going to access LDAP, use mysql to store the username and password, you can ignore the following configuration
# for ldap authorization
ldap:
  host: "ldap.example.org"
  port: 389
  baseDn: "dc=example,dc=org"
  bindUser: "cn=manager,dc=example,dc=org"
  bindPass: "*******"
  # openldap: (&(uid=%s))
  # AD: (&(sAMAccountName=%s))
  authFilter: "(&(uid=%s))"
  tls: false
  startTLS: false

# Depends on which alarm channels you plan to connect to, if it is just email, just keep the default configuration
# p1 /p2 /p3 represents the alarm level. In many large companies, different alarm levels correspond to different alarm channels
# p1 is the highest priority alarm, so it will be sent using all channels, p3 is the lowest priority, only send mail and im
# After configuring different sending methods here, each method needs to deploy the corresponding sender module
# If the sender module is not deployed, but the relevant configuration is opened here, it will cause a message backlog in redis, monapi will push different alarm messages to different redis queues, and send it to the sender for consumption
# notify support: voice, sms, mail, im
# if we have all of notice channel
# notify:
#   p1: ["voice", "sms", "mail", "im"]
#   p2: ["sms", "mail", "im"]
#   p3: ["mail", "im"]

# if we only have mail channel
notify:
  p1: ["mail"]
  p2: ["mail"]
  p3: ["mail"]

# There will be some hyperlinks in the alarm message, click it to quickly access the policy page and event page corresponding to the alarm
# Here mainly modify n9e.example.com to your platform domain name, or the platform's nginx ip: port
# If it is ip: port, it needs to be reachable by the local browser, can not use idc intranet ip
# addresses accessible using browsers
link:
  stra: http://n9e.example.com/#/monitor/strategy/%v
  event: http://n9e.example.com/#/monitor/history/his/%v
  claim: http://n9e.example.com/#/monitor/history/cur/%v

# for alarm event and message queue
redis:
  addr: "127.0.0.1:6379"
  pass: ""
  # in ms
  # timeout:
  #   conn: 500
  #   read: 3000
  #   write: 3000  
```

When monapi is deployed and packaged, you can package the pub directory together, which is the front-end resource file. Nginx is generally mixed with monapi, nginx depends on the pub directory.

### index
index is an index processing module. You can deploy 2 ~ 3 instances, send heartbeat information to monapi, and do not depend on other components. It can be deployed after the monapi deployment is completed. The relevant configuration is as follows:

```yaml
logger:
  dir: logs/index
  level: WARNING
  keepHours: 2

# identity is the unique identity reported to monapi, tsdb uses identity to obtain the index address list
# You can manually set the identity through specify. If specify is empty, it will be automatically obtained by executing a shell command. The default is to obtain the IP
identity:
  specify: ""
  shell: ifconfig `route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|awk -F ':' '{print $NF}'|head -n 1

# The index module is mainly used to save the index information of the monitoring system. The following is the relevant configuration of the index information, the system will have a default configuration, generally do not need to care.
# When the index restarts, the index data in the memory will be lost, and the tsdb module caches the index sent to the index, and it will not be sent again in a short time. This will cause the previous index to be missing after the index restarts, so it needs to be in memory. Persistence of the index, persistDir defaults to ./.index
# If the index is terminated unexpectedly, it is too late to take the initiative to place the data, and the index data will also be lost, so there is a need to have a mechanism to periodically persist the data. The default interval (persistInterval) is 15 minutes
# cache:
#   persistInterval: 900
#   persistDir: "./.index"
```

The binary files for index deployment are n9e-index, etc /index.yml, etc /address.yml.

### tsdb

tsdb is a data storage module that forwards a copy of data to index, so it is deployed after index. Why is the data of index not forwarded directly by transfer, but forwarded by tsdb? Because the index data does not need to be updated at all times, only incremental updates are required. To distinguish between upstream and stock, upstream needs to cache the stock (which has been sent to index) to distinguish which data has been sent to index and which has not. If this cache is placed in the transfer module, transfer needs to cache all the data on the platform, and tsdb is fragmented. If tsdb is used for caching, only the data of its own fragment needs to be cached, and the memory footprint is less.

If tsdb is a 16C32G configuration and uses ssd, it can easily resist the writing of 10,000 monitoring indicators per second. If there are many devices to be monitored, multiple tsdb can be deployed to form a cluster to share the pressure. If you are worried that a tsdb cluster will hang, you can deploy two clusters at the same time to allow transfer to double-write.

```yaml
logger:
  dir: logs/tsdb
  level: WARNING
  keepHours: 2

# rrdtool related configuration, storage is the directory where the rrd file is located
rrd:
  storage: data/5821

# The most recently reported monitoring data is stored in tsdb memory to improve query efficiency. KeepMinutes is the length of time to save the latest time series data, the default is 2 hours
# cache:
#   keepMinutes: 120
```

The files that tsdb deployment depends on are n9e-tsdb, etc /tsdb.yml, etc /address.yml.

### judge

Judge is an alarm engine and naturally requires an alarm strategy. The judge relies on monapi to pull the alarm strategy, and judges whether the monitoring data triggers the alarm threshold according to the alarm strategy, and the generated alarm event is pushed to redis. If redis is highly available, you can use one redis globally. If not, you can deploy two standalone versions of redis, then configure the two redis addresses in the judge, and then deploy two of monapi, one by one with redis correspond.

The judge can deploy multiple for disaster recovery and periodically heartbeat to monapi. The configuration of judge is as follows:

```yaml
logger:
  dir: logs/judge
  level: WARNING
  keepHours: 2

# The judge needs to query data and indexes from tsdb when doing nodata and conditional alarms. The following are some timeout settings for querying data and indexes, in milliseconds
query:
  # in ms
  connTimeout: 1000
  callTimeout: 2000
  indexCallTimeout: 2000

# for alarm event and message queue
redis:
  addrs: 
    - 127.0.0.1:6379
  pass: ""
  # in ms
  # timeout:
  #   conn: 500
  #   read: 3000
  #   write: 3000

# identity is the unique identifier reported to monapi, monapi will issue a strategy to judge based on the identity reported by judge
# Can be set manually by specify, if specify is empty, it will be automatically obtained by executing a shell command, the default is to obtain the IP
identity:
  specify: ""
  shell: ifconfig `route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|awk -F ':' '{print $NF}'|head -n 1
```

The files that judge's deployment depends on are n9e-judge, etc /judge.yml, etc /address.yml.

### transfer

Transfer is a data receiving and forwarding component. Transfer receives the data pushed by the collector, and also pushes some custom collectors, and then forwards a copy of the data to tsdb, and at the same time forwards a copy to the judge. The address of the judge is obtained by asking monapi, and the address of tsdb is directly configured in the transfer configuration file. Because tsdb is a data storage module, it is very important, and some instances cannot be easily dropped because the network is not available. This will cause data to be redistributed, causing some old data to be invisible. In addition, transfer is stateless, and multiple instances can be deployed horizontally. The configuration of transfer is as follows:

```yaml
# Transfer needs to query data from tsdb, and backend is related configuration of query operation
# timeout is the timeout setting for querying data in milliseconds
# cluster is the address list of tsdb cluster, if tsdb is a double write cluster, the configuration is as follows, the two addresses are separated by `,`
# cluster:
#   tsdb01: cluster1.addr:5821,cluster2.addr:5821
backend:
  # in ms
  # connTimeout: 1000
  # callTimeout: 3000
  cluster:
    tsdb01: 127.0.0.1:5821 

logger:
  dir: logs/transfer
  level: WARNING
  keepHours: 2
```

The transfer dependent files are n9e-transfer, etc /transfer.yml, etc /address.yml.

### collector

The collector is a monitoring data collection and forwarding component that needs to be deployed on every machine that needs to be monitored. Collector collects and reports the system, port, process and log information of the machine to the transfer according to the user's configuration in monapi and local files. It also receives some custom collector pushes.

```yaml
logger:
  dir: logs/collector
  level: WARNING
  keepHours: 2

# The identity is the endpoint reported to the transter monitoring data, which can be manually set by specify. If specify is empty, it will be automatically obtained by executing a shell command. The default is to obtain the IP
identity:
  specify: ""
  shell: ifconfig `route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|awk -F ':' '{print $NF}'|head -n 1

sys:
  # timeout in ms
  # interval in second
  timeout: 1000
  # interval is the reporting period of the collection system monitoring indicators, the default is to collect and report once every 20s
  interval: 20
  # If you want to collect a certain type of network card, you can write its prefix below
  ifacePrefix:
    - eth
    - em
  # If you want to collect a certain mount point, you can write it below, if it is empty, it means all collection
  mountPoint: []
  # Some mount point collection is meaningless, you can write its prefix to the following list
  mountIgnorePrefix:
    - /var/lib
  # If some monitoring indicators do not want to be collected, you can write to the following
  ignoreMetrics: 
    - cpu.core.idle
    - cpu.core.util
    - cpu.core.sys
    - cpu.core.user
    - cpu.core.nice
    - cpu.core.guest
    - cpu.core.irq
    - cpu.core.softirq
    - cpu.core.iowait
    - cpu.core.steal
```

Collector deployment dependent files are n9e-collector, etc /collector.yml, etc /address.yml.
The address.yml here may not be easy to understand, and a video was specifically recorded：[explain address.yml](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-usage-address.mp4) Put these three files in the /home /n9e directory of the target machine, then modify the monapi and transfer addresses fields in etc /address.yml, and configure them as the real server ip address。Because the collector communicates with transfer and monapi, it must know the ip addresses of these two components. If you still do n’t understand, watch this video：[collector单独部署](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-install-collector.mp4)

### nginx

As a reverse proxy, nginx proxies front-end requests for monapi, transfer, and index according to different locations. At the same time, the front-end resources under the pub directory are also served by nginx, and the nginx configuration file can be found in etc /nginx.conf.

### sender

By Reading《[Compile and install through source code](../compile/)》，sender can see[mail-sender](https://github.com/n9e/mail-sender)，Looking forward to writing wechat-sender, dingding-sender, sms-sender, slack-sender, and building a community :-)