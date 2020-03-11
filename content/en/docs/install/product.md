
---
title: "生产环境分布式安装"
linkTitle: "生产安装"
date: 2020-02-24
description: >
  本节主要介绍各个模块如何做分布式安装，适用于正式生产环境，服务稳定性很重要，故而及时发现问题的监控系统很重要，多用几台机器提高容灾能力非常有必要
---


为了容灾考虑，生产环境部署Nightingale，我们一般至少采用2台机器（mysql、redis单独做高可用部署，不在这2台机器上混部），如果您要监控的设备、服务比较多，需要适当增加机器，对于1000台左右设备的场景，最好有至少2台16C/32G/300G SSD以上配置的机型。

在《[源码编译安装](../compile/)》一节，已经介绍过如何拿到二进制发布包，如何导入数据库，本节不再赘述；mysql、redis也假设已经搭建好了；本节主要讲解各模块的关联性配置部分。

### 架构回顾

这里把架构图再贴在这里一份，以供参考，了解了架构再来搭建事半功倍，我们会挨个模块介绍其大体功能和配置要点。

![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-arch.png)

### 写在前面

既然是生产环境，所有模块挂掉都要自动拉起，都要设置开机自启动，可以使用systemd或者supervisor之类的托管，在etc/service目录下可以找到各模块的service文件示例，供参考。另外，所有依赖mysql的模块，都统一读取mysql.yml，故而，mysql的连接配置只会出现在mysql.yml。

### 关联配置

每个模块都有自己监听的端口，都跟别的模块有关系，要各种关联访问，我们统一提取一个address.yml的配置文件，配置各个模块listen的地址，以及各个模块部署的机器列表用于给调用方使用。所以，所有的模块单独部署的时候，不但要带上自己同名的配置文件，还要带上address.yml

### monapi

主要承接web端的请求，无状态，可以水平扩展部署多台，依赖mysql和redis，请提前把密码准备好。monapi要run起来依赖的配置文件有3个，除了address.yml，另外就是mysql.yml和monapi.yml，mysql.yml里是数据库连接配置，一眼就懂，不啰嗦，这里主要说一下monapi.yml

```yaml
---
# 这是密码加密时使用的随机字符串，自行给一个随机串即可
salt: "PLACE_SALT"

logger:
  dir: "logs/portal"
  level: "WARNING"
  keepHours: 24

# secret是加密cookie的随机字符串，自行给一个随机串即可
http:
  secret: "PLACE_SECRET"

# LDAP验证相关配置，请协调LDAP管理员帮忙配置，他们懂
# 如果您不准备接入LDAP，使用mysql存储用户名和密码则可忽略下面的配置
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

# 看贵司准备接入哪些告警通道，如果只是邮件则维持默认配置即可
# p1/p2/p3代表告警级别，一般大厂，不同告警级别对应不同的告警通道
# p1是最高优的告警，所以会利用所有通道发出，p3是最低优的，只发邮件和im即可
# 这里配置了不同的发送方式之后，每个方式都需要部署对应的sender模块
# 如果没有部署sender模块，却在这里打开了相关配置，那会在redis里造成
# 消息积压，monapi会把不同的告警消息推给不同的redis队列，交由sender来消费
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

# 告警消息中会带有一些超链接，点击可以快速访问到告警对应的策略页面、事件页面等
# 这里主要修改n9e.example.com，修改成您的平台域名，或者平台的nginx的ip:port
# 如果是ip:port，需要是本地浏览器可达的ip，不能使用idc内网ip
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

monapi部署打包的时候可以把pub目录一并打包，这是前端资源文件，nginx一般和monapi混部，nginx依赖pub目录。

### index

index是索引处理模块，可以部署2~3个实例，会发心跳信息给monapi，不依赖其他组件，可以在monapi部署完成之后来部署，相关配置如下：

```yaml
logger:
  dir: logs/index
  level: WARNING
  keepHours: 2

# identity为上报给monapi的唯一标识，tsdb获取index地址列表的时候使用
# 可以通过specify手动设置，如果specify为空，则会通过执行shell命令自动获取，默认是获取IP 
identity:
  specify: ""
  shell: /usr/sbin/ifconfig `/usr/sbin/route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|head -n 1

# index模块主要是保存监控系统的索引信息，下面是索引信息的相关配置，系统会有一个默认配置，一般可以不用关心
# 当index重启的时候，内存的索引数据就会丢失，而tsdb模块对发给过index的索引有缓存，短期不会重复发送，这样会导致index重启之后，之前的索引缺失，所以需要对内存中的索引持久化，持久化数据所在的目录(persistDir)默认为 ./.index
# 如果index意外终止，来不及主动将数据落盘，索引数据也会丢失，所以需要有一个定期持久化数据的机制，默认周期(persistInterval)为15分钟
# cache:
#   persistInterval: 900
#   persistDir: "./.index"
```

index的部署依赖的文件是n9e-index二进制、etc/index.yml、etc/address.yml。

### tsdb

tsdb是数据存储模块，会转发一份数据给index，所以在index之后部署，为何index的数据不直接由transfer转发，而是由tsdb转发？是因为索引数据无需时刻更新，只需要增量更新即可，上游要想区分是增量还是存量，需要对存量（已经发送给index的）做缓存，来区分哪些数据已经发给index了，哪些还没有，如果这个缓存放在transfer模块，transfer就需要缓存平台上的所有数据，而tsdb是分片的，tsdb来做缓存的话，只需要缓存自己分片的数据，内存占用较少。

tsdb如果使用ssd的机器，16C32G的配置，可以轻松抗住每秒1万监控指标的写入，如果要监控的设备很多，可以部署多个tsdb组成集群分担压力。如果担心一个tsdb集群挂掉，可以同时部署2个集群，让transfer来双写，相关配置如下：

```yaml
logger:
  dir: logs/tsdb
  level: WARNING
  keepHours: 2

# rrdtool相关配置，storage为rrd文件所在的目录
rrd:
  storage: data/5821

# 最近上报的监控数据都保存在tsdb内存中，以此来提高查询效率，keepMinutes为保存最新时序数据的时长，默认为2个小时
# cache:
#   keepMinutes: 120
```

tsdb的部署依赖的文件是n9e-tsdb二进制、etc/tsdb.yml、etc/address.yml。

### judge

judge是告警引擎，自然需要告警策略，依赖monapi来拉取告警策略，根据告警策略判断监控数据是否触发告警阈值，生成的告警事件推给redis，这里redis如果是高可用的，全局就用一个redis即可，如果不是，可以部署2个单机版本的redis，然后在judge中配置这两个redis的地址，然后monapi也部署两个，与redis一一对应，一个monapi来消费一个redis的数据。

judge为了容灾考虑可以部署多个，会周期性向monapi心跳，judge的配置如下：

```yaml
logger:
  dir: logs/judge
  level: WARNING
  keepHours: 2

# judge在做nodata和与条件告警的时候，需要从tsdb中查询数据和索引，下面是查询数据和索引的一些超时设置，单位为毫秒
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

# identity为上报给monapi的唯一标识，monapi会根据judge上报的identity给judge下发策略
# 可以通过specify手动设置，如果specify为空，则会通过执行shell命令自动获取，默认是获取IP 
identity:
  specify: ""
  shell: /usr/sbin/ifconfig `/usr/sbin/route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|head -n 1
```

judge的部署依赖的文件是n9e-judge二进制、etc/judge.yml、etc/address.yml。

### transfer

transfer是个数据接收、转发组件，接收collector推送的数据，也接收一些自定义采集器的推送，然后将数据转发一份给tsdb，转发一份给judge，judge的地址是通过询问monapi获得，tsdb的地址是直接配置在transfer的配置文件中的，因为tsdb是数据存储模块，非常重量级，不能因为网络不通就轻易踢掉部分实例，这会导致数据重新分配，导致部分老数据就看不到了。另外，transfer是无状态的，可以水平扩展部署多个实例，transfer的配置如下：

```yaml
# transfer需要从tsdb中查询数据，backend为查询操作的相关配置
# timeout是查询数据的超时设置，单位为毫秒
# cluster是tsdb集群的地址列表，如果tsdb为双写集群，则配置如下，两个地址之间通过逗号隔开
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

transfer的部署依赖的文件是n9e-transfer二进制、etc/transfer.yml、etc/address.yml。

### collector

collector是个监控数据采集、转发组件，需要部署在每一台需要监控的机器上，根据用户在monapi和本地文件的配置，对机器的系统、端口、进程和日志信息进行采集并上报给transfer，也接收一些自定义采集器的推送，collector的配置如下：

```yaml
logger:
  dir: logs/collector
  level: WARNING
  keepHours: 2

# identity为上报给transter监控数据中的endpoint，可以通过specify手动设置，如果specify为空，则会通过执行shell命令自动获取，默认是获取IP 
identity:
  specify: ""
  shell: /usr/sbin/ifconfig `/usr/sbin/route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|head -n 1

sys:
  # timeout in ms
  # interval in second
  timeout: 1000
  # interval为系统监控指标的采集上报周期，默认是20s采集上报一次
  interval: 20
  # 如果想要采集某个类型的网卡，可以将其前缀写到下面
  ifacePrefix:
    - eth
    - em
  # 如果想要采集某个挂载点，可以将其写到下面，如果为空，则表示全部采集
  mountPoint: []
  # 有的挂载点采集没有意义，可以将其前缀写到下面的列表
  mountIgnorePrefix:
    - /var/lib
  # 如果某些监控指标不想采集，则可以写到下面
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

collector的部署依赖的文件是n9e-collector二进制、etc/collector.yml、etc/address.yml。

### nginx

nginx作为反向代理，根据不同location代理了monapi、transfer、index的前端请求，同时，pub目录下的前端资源也是由nginx来serve，nginx的配置文件参看etc/nginx.conf，这里不再赘述

### sender

如《[通过源码编译安装](../compile/)》一节所述，sender可以参看[mail-sender](https://github.com/n9e/mail-sender)，期待您编写wechat-sender、dingding-sender、sms-sender、slack-sender，共建社区 :-)