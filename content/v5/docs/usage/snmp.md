---
title: "SNMP"
linkTitle: "SNMP"
date: 2021-05-21
description: >
  夜莺的客户端(n9e-agentd)中内置了SNMP采集器，可以采集各类网络设备的监控指标。一般我们会在不同的机房分别找一个机器作为探针，开启这个机器的n9e-agentd的SNMP采集能力，采集本机房的网络设备
---

## 概述
简单网络管理协议 (SNMP)是用于监视网络连接设备（例如路由器、交换机、服务器和防火墙）的标准。从网络设备收集 SNMP 指标。

SNMP 使用 sysObjectID（系统对象标识符）来唯一标识设备，使用 OID（对象标识符）来唯一标识管理对象。OID 遵循分层树模式：根下是 ISO，编号为 1。下一级是 ORG，编号为3，依此类推，每个级别之间用 . 分隔。

MIB（Management Information Base）充当 OID 和人类可读名称之间的转换器，并组织层次结构的子集。由于树的结构方式，大多数 SNMP 值都以相同的对象集开头：

- 1.3.6.1.1: (MIB-II) 一种保存系统信息（如正常运行时间、接口和网络堆栈）的标准。
- 1.3.6.1.4.1：保存供应商特定信息的标准。

## 安装
 [n9e-agentd](https://github.com/n9e/n9e-agentd) 代理程序中已经包含 SNMP 采集器，无需额外安装。

## 配置
agentd 网络设备监控支持从单个设备收集指标，或自动发现整个子网上的设备（网络内设备的IP必须唯一）。

可以根据网络上存在的设备数量以及网络的动态程度（添加或删除设备的频率）选择适合的采集策略：

- 对于小型且大部分为静态的网络，请参阅 [监控单个设备](#监控单个设备)。
- 对于更大的或动态的网络，请参阅 [自动发现](#自动发现)。

无论采用何种采集策略，都可以利用 agentd 的 sysObjectID 映射设备配置文件自动从您的设备收集相关指标。


##### 监控单个设备

将 IP 地址和任何其他设备元数据（作为标签）snmp.d/conf.yaml包含在代理配置目录conf.d/根文件夹中的文件中。有关所有可用的配置选项，请参阅示例 snmp.d/conf.yaml。

1. SNMPv2
```yaml
# /opt/n9e/agentd/conf.d/snmp.d/conf.yaml
init_config:
  loader: core
instances:
- ip_address: "1.2.3.4"
  community_string: "public"
  # tags:
  #   - "region:beijing"
  #   - "usefor:firewall"
```

2. SNMPv3
```yaml
# /opt/n9e/agentd/conf.d/snmp.d/conf.yaml
init_config:
  loader: core
instances:
- ip_address: "1.2.3.4"
  snmp_version: 3			# optional, if omitted we will autodetect which version of SNMP you are using
  user: "user"
  auth_protocol: "fakeAuth"
  auth_key: "fakeKey"
  # priv_protocol:
  # priv_key:
  # tags:
  #   - "region:shanghai"
  #   - "usefor:switch"
```

- 重启 agentd
```bash
sudo systemctl restart n9e-agentd
```

##### 自动发现

指定单个设备的替代方法是使用 Autodiscovery 来自动发现网络上的所有设备。

自动发现配置的网段，并检查来自目标设备的响应。然后，agentd 代理查找 sysObjectID 发现的设备并找到设备对应的  [配置文件](https://github.com/n9e/n9e-agentd/tree/main/misc/conf.d/snmp.d/profiles)。配置文件中包含各种设备收集的预定义指标列表。

将自动发现与网络设备监控结合使用：
- 编辑 agentd.yaml 配置文件，设置 要扫描的所有子网。以下示例提供了自动发现所需的参数、默认值。

1. SNMPv2
```yaml
# /opt/n9e/agentd/etc/agentd.yaml
agent:
  listeners:
    - name: snmp
  snmp_listener:
    workers: 100 # number of workers used to discover devices concurrently
    discovery_interval: 3600 # interval between each autodiscovery in seconds
    configs:
      - network: 1.2.3.4/24 # CIDR notation, we recommend no larger than /24 blocks
        version: 2
        port: 161
        community: "public"
        # tags:
        #   - "region:beijing"
        #   - "usefor:firewall"
        loader: core # use SNMP corecheck implementation
      - network: 2.3.4.5/24
        version: 2
        port: 161
        community: "public"
        # tags:
        #   - "region:beijing"
        #   - "usefor:switch"
        loader: core
```

2. SNMPv3
```yaml
# /opt/n9e/agentd/etc/agentd.yaml
agent:
  listeners:
    - name: snmp
  snmp_listener:
    workers: 100 # number of workers used to discover devices concurrently
    discovery_interval: 3600 # interval between each autodiscovery in seconds
    configs:
      - network: 1.2.3.4/24 # CIDR notation, we recommend no larger than /24 blocks
        version: 3
        user: "user"
        authentication_protocol: "fakeAuth"
        authentication_key: "fakeKey"
        # privacy_protocol:
        # privacy_key:
        # tags:
        #   - "region:beijing"
        #   - "usefor:firewall"
        loader: core
      - network: 2.3.4.5/24
        version: 3
        snmp_version: 3
        user: "user"
        authentication_protocol: "fakeAuth"
        authentication_key: "fakeKey"
        # privacy_protocol:
        # privacy_key:
        # tags:
        #   - "region:beijing"
        #   - "usefor:firewall"
        loader: core
```

**注意**：agentd 会自动发现设备IP，然后依次采集每一个正常应答的设备。


## profile 配置
SNMP profile为某些品牌和型号的网络设备提供开箱即用监控的配置, 目录 /opt/n9e/agentd/conf.d/snmp.d/profiles.

文件结构如下 

```yaml
sysobjectid: <x.y.z...>

# extends:
#   <Optional list of base profiles to extend from...>

metrics:
  # <List of metrics to collect...>

# metric_tags:
#   <List of tags to apply to collected metrics. Required for table metrics, optional otherwise>
```

下面的例子中, 查询 172.25.79.194 的 `sysobjectid` 为  `1.3.6.1.4.1.25506.1.1210`
```Bash
$ snmpwalk -On 172.25.79.194 1.3.6.1.2.1.1.2.0
.1.3.6.1.2.1.1.2.0 = OID: .1.3.6.1.4.1.25506.1.1210
```

采集器的配置文件 /opt/n9e/agentd/conf.d/snmp.d/conf.yaml 中,  采集器会首先查询目标的 sysobjectid, 在 profile 目录查找相匹配的配置, 也可以直接在 conf.yaml 文件中配置 metrics 和 metric_tags, 格式和 profile 一样

### 字段

#### `sysobjectid`

_(必须)_

`sysobjectid` 字段用于在设备自动发现期间将配置文件与设备进行匹配。

它可以引用特定设备品牌和型号的完全定义的 OID：

```yaml
sysobjectid: 1.3.6.1.4.1.232.9.4.10
```

或通配符模式来解决多个设备模型：

```yaml
sysobjectid: 1.3.6.1.131.12.4.*
```

或完全定义的 OID / 通配符模式列表：

```yaml
sysobjectid:
  - 1.3.6.1.131.12.4.*
  - 1.3.6.1.4.1.232.9.4.10
```

#### `metrics`
 参考 [profiles](https://datadoghq.dev/integrations-core/tutorials/snmp/profiles/)

#### `metrics_tags`
 参考 [profiles](https://datadoghq.dev/integrations-core/tutorials/snmp/profiles/)


## FAQ

#### 自动发现不起作用?

确认agentd是否为最新版本。确认方法是看这个文件是否存在：/opt/n9e/agentd/conf.d/snmp.d/auto_conf.yaml 官网最新安装包里是有这个文件的，或者去agentd的github release页面下载，[下载地址](https://github.com/n9e/n9e-agentd/releases)

## Resources
- https://docs.datadoghq.com/network_monitoring/devices/setup
- https://datadoghq.dev/integrations-core/tutorials/snmp/profiles/
- http://cric.grenoble.cnrs.fr/Administrateurs/Outils/MIBS/
- http://circitor.fr/Mibs/Mibs.php
- http://mibs.snmplabs.com/asn1/
- https://en.wikipedia.org/wiki/Simple_Network_Management_Protocol
- https://www.youtube.com/playlist?list=PL4j_fCKQ7Bso1wGplcaF2pcDGeLVCuKHU
- https://en.wikipedia.org/wiki/Field-replaceable_unit
