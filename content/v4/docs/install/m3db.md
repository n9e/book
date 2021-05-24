
---
title: "存储切换M3方式，强烈建议"
linkTitle: "存储切换M3方式"
date: 2020-02-19
description: >
  夜莺在v3版本引入了m3作为后端存储，m3在uber声称存储了66亿监控指标，非常强悍，建议尝试
---


初期可以使用单机版m3db做测试，有了解之后上生产建议使用集群版。用了m3之后，原来的index模块和tsdb模块就可以不用了。m3相比原来的index+tsdb的方案，优劣势是什么？

优点：

- 对硬盘IO要求没那么高了，普通机械硬盘也能抗比较大的量
- 存原始数据，不降采样了，追问题的时候更方便，当然了，存储的数据时长就变短了，相同硬盘空间大小，比如原来rrd可以存1年的历史趋势数据，m3可能只能存1个月
- 扩容非常方便，直接加m3dbnode即可，index+tsdb的方案使用migrate配置，扩容不易
- 容灾更好了，可以设置3副本，如果集群部署了3台机器，挂掉一台机器完全没有影响
- 索引避免了原来index+tsdb的单点容量问题，原来index虽然是可以部署为集群，但是集群里每个节点都是全量索引

劣势：

- 硬盘空间占用大，毕竟存储原始数据嘛，一般生产环境建议存1个月，再久也尽量不要超过3个月，当然了，要监控的设备比较少，部署m3的机器硬盘又比较大，那另当别论
- 内存占用比较多，一般配置是最近两个小时的数据要缓存在内存里，所以比较吃内存，好在内存现在也便宜了，一台机器动不动128G、256G的

# 通用改动

即不管是单机版m3还是集群版，对于n9e来说都要修改的一些配置。

## 修改 transfer 配置

```yaml
backend:
  datasource: "m3db"  # 选择 m3db 为索引和存储源
  m3db:
    enabled: true  # 开启 m3db
    name: "m3db"
    namespace: "test"
    seriesLimit: 0
    docsLimit: 0
    daysLimit: 7                               # 查询时的最大时间限制
    # 读写数据时一致性检查的等级，请参考官方文档
    # https://m3db.github.io/m3/m3db/architecture/consistencylevels/
    writeConsistencyLevel: "majority"          # one|majority|all
    readConsistencyLevel: "unstrict_majority"  # one|unstrict_majority|majority|all
    config:
      service:
        # KV environment, zone, and service from which to write/read KV data (placement
        # and configuration). Leave these as the default values unless you know what
        # you're doing. 请与m3.dbnode 的配置保持一致
        env: default_env
        zone: embedded
        service: m3db
        # 以下是 etcd 的配置
        etcdClusters:
          - zone: embedded
            # 单机版的话，endpoints这部分一定要配置为127.0.0.1的
            endpoints:
              - 127.0.0.1:2379
            # 单机版不需要搞证书，单机版的etcd是内嵌在m3dbnode里的，把tls相关的这4行注释掉
            tls:
              caCrtPath: /opt/run/etcd/certs/root/ca.pem
              crtPath: /opt/run/etcd/certs/output/etcd-client.pem
              keyPath: /opt/run/etcd/certs/output/etcd-client-key.pem
```

## 修改 monapi 配置

```yaml
# 新增 indexMod 配置
indexMod: transfer
logger:
  dir: logs/monapi
  level: INFO
  keepHours: 24
...
```

## 修改 judge 配置

```yaml
query:
  indexMod: transfer
  connTimeout: 1000
  callTimeout: 2000
  indexCallTimeout: 2000
...
```

## 修改nginx配置

使用M3DB替代之前的自研index+rrd的方案，原来的index模块和tsdb模块就可以下掉了，nginx里index相关的配置需要改动，原来的配置：

```
location /api/index {
    proxy_pass http://n9e.index;
}
```

改成：

```
location /api/index {
    proxy_pass http://n9e.transfer;
}
```

相当于，使用M3DB做后端的存储的话，transfer模块会承接索引请求，转发给M3处理


## M3官方资料

- [installation](https://operator.m3db.io/getting_started/installation/)
- [single_node](https://m3db.github.io/m3/how_to/single_node/)
- [cluster_hard_way](https://m3db.github.io/m3/how_to/cluster_hard_way/)

建议大伙尽量还是去官方了解一下M3，uber声称他们用m3存储了66亿指标，真的非常牛逼。如果不想深究，想先用起来测试，请参考下面的方法

## 单机版M3

这个试用于单机测试场景，所有夜莺服务端组件都部署在一台机器上，特别是transfer和m3要部署在一台机器上的场景，用来测试或者应对小规模场景，足够了

### 安装方法

单机安装包[m3dbnode-single-v0.0.1.tar.gz](https://s3-gz01.didistatic.com/n9e-pub/tarball/m3dbnode-single-v0.0.1.tar.gz) ，解包之后有个README是安装文档，主要是两个操作步骤：

1、解包之后执行./scripts/install.sh，这个脚本在centos7上面测试ok，用了systemd的方式托管进程，大家可以打开看看脚本内容
2、调用m3的接口，创建`/api/v1/database/create`创建database，README里有个例子，retentionTime是数据存储时长，测试阶段可以设置为48h，存2天，足够了

### 检查状态

因为是systemd托管的，所以 `systemctl status m3dbnode` 即可查看进程状态。机器重启之后如果希望进程自动拉起，记得执行 `systemctl enable m3dbnode`

## 集群版M3

[m3db-install-v0.0.2.tar.gz](https://s3-gz01.didistatic.com/n9e-pub/tarball/m3db-install-v0.0.2.tar.gz) 

### 配置

#### etcd证书

```shell
# 修改 ./etcd/certs/config/hosts-etcd
# 填写etcd的机器列表

# 生成证书
cd etcd/certs
./reinit-root-ca.sh
./update-etcd-client-certs.sh
./update-etcd-server-certs.sh
```


#### m3db配置

```shell
# 修改 ./m3db/etc/m3dbnode.yml
db:
  config:
    service:
      etcdClusters:
        - zone: embedded_env
          endpoints:
            - 10.255.0.146:2379  # 这里需要修改 etcd 节点信息
            - 10.255.0.129:2379
```

#### 分发安装

为了避免麻烦，用来发起m3安装的机器，和目标的3台机器尽量不要复用

```shell
. ./functions.sh

# set hosts
hosts="{node1} {node2} {node3}"

# sync etcd,m3db to hosts
sync

# install
install

# status
status
```

#### 安装之后的路径

```
# database path
/home/etcd
/home/m3db
/home/m3kv

# config path
/etc/etcd/
/etc/m3db/
```

#### 初始化 m3dbnode database

修改 ./m3db/scripts/01-create-db.sh 后，执行：

```shell
curl -X POST http://localhost:7201/api/v1/database/create -d '{
  "type": "cluster",
  "namespaceName": "default",
  "retentionTime": "2160h",
  "numShards": "64",
  "replicationFactor": "3",
  "hosts": [
        {
            "id": "10-255-0-146",
            "isolationGroup": "group1",
            "zone": "embedded",
            "weight": 100,
            "address": "10.255.0.146",
            "port": 9000
        },
        {
            "id": "10-255-0-129",
            "isolationGroup": "group2",
            "zone": "embedded",
            "weight": 100,
            "address": "10.255.0.129",
            "port": 9000
        },
        {
            "id": "10-255-0-137",
            "isolationGroup": "group3",
            "zone": "embedded",
            "weight": 100,
            "address": "10.255.0.137",
            "port": 9000
        }
    ]
}'
```
- hosts[0].id: 这是机器节点的标识，默认情况下，用的是机器的hostname； 取决于m3dbnode.yml: db.hostID.resolver如何配置
- retentionTime: 数据保存时长，默认是90天，磁盘空间不够的话，建议改成8天，即192，不是7天，是担心后面可能有周环比的看图需要
- numShards: 如果是单机测试，这里改成8，如果是3台机器可以用64，如果有更多机器，可以参照下一章节来定义这个数量
- replicationFactor: 如果单节点就改成1即可


### 扩展阅读

节点数量， 建议至少3台机器，数据复制3份(RF)，参考  [replication_and_deployment_in_zones](https://m3db.github.io/m3/operational_guide/replication_and_deployment_in_zones/)

Replica Factor | Quorum Size | Failure Tolerance
--             | --          | --
1              | 1           | 0
2              | 2           | 0
3              | 2           | 1
4              | 3           | 1
5              | 3           | 2
6              | 4           | 2
7              | 4           | 3



数据分片数量，根据未来的规模配置，参考 [placement_configuration](https://m3db.github.io/m3/operational_guide/placement_configuration/)


Number of Nodes | Number of Shards
--              | --
3               | 64
6               | 128
12              | 256
24              | 512
48              | 1024
128+            | 4096


