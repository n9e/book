
---
title: "安装部署"
linkTitle: "安装部署"
weight: 3
date: 2021-06-23
description: >
  阅读本章节，一定要先阅读《[夜莺介绍](/docs/intro/)》，了解整体的架构，这样出了问题方能有思路排查。因为大部分生产环境都是使用CentOS7，后面的讲解我们都将基于此发行版展开，所有二进制都是X86架构下编译而成，这点请注意。
---

> 演示环境：[http://116.85.46.86/](http://116.85.46.86/) 用户名：demo，密码：demo.2021

回顾《[夜莺介绍](/docs/intro/)》中的架构讲解，整个数据流是`agentd->server<->storage`，即agentd采集监控数据，推送给server，server会做各种处理，然后交由存储来持久化数据，后续server要读取数据的时候就从存储来读。所以，我们的部署顺序就是从被依赖方开始部署：1.部署存储 2.部署服务端 3.部署客户端

## 部署存储

存储是可插拔的方式，简单起见，我们选择单机版Prometheus来快速开始，部署Prometheus的方式如下：

```bash
mkdir -p /opt/prometheus

wget https://s3-gz01.didistatic.com/n9e-pub/prome/prometheus-2.28.0.linux-amd64.tar.gz -O prometheus-2.28.0.linux-amd64.tar.gz
tar xf prometheus-2.28.0.linux-amd64.tar.gz
cp -far prometheus-2.28.0.linux-amd64/*  /opt/prometheus/

# service 
cat <<EOF >/etc/systemd/system/prometheus.service
[Unit]
Description="prometheus"
Documentation=https://prometheus.io/
After=network.target

[Service]
Type=simple
ExecStart=/opt/prometheus/prometheus  --config.file=/opt/prometheus/prometheus.yml --storage.tsdb.path=/opt/prometheus/data --web.enable-lifecycle --enable-feature=remote-write-receiver --query.lookback-delta=2m 

Restart=on-failure
RestartSecs=5s
SuccessExitStatus=0
LimitNOFILE=65536
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=prometheus


[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable prometheus
systemctl restart prometheus
systemctl status prometheus
```

## 部署服务端

服务端依赖mysql数据库，请自行安装，v5版本的数据库表结构和之前的版本不兼容，所以没法复用之前版本的数据库，这点请注意。

#### 1. 下载安装包

```bash
mkdir -p /opt/n9e
cd /opt/n9e
wget 116.85.64.82/n9e-5.0.0-rc1.tar.gz
tar zxvf n9e-5.0.0-rc1.tar.gz
```

#### 2. 导入表结构

```
mysql -uroot -p < /opt/n9e/sql/n9e.sql
```

#### 3. 修改配置

服务端启动的时候会看etc目录下是否有server.local.yml，如果有就用，如果没有，再去找server.yml，即server.local.yml的优先级高于server.yml

```
cd /opt/n9e/etc
cp server.yml server.local.yml
# 修改server.local.yml中的数据库连接配置
# 默认配置的后端存储就是Prometheus，所以不用改动
```

#### 4. 启动进程

```
cd /opt/n9e
nohup ./n9e-server &> server.log &

# 进程如果启动了，理论上会监听2个端口，一个http端口一个rpc端口
# 通过下面命令可以查看端口是否在监听，如果端口都在监听，就说明启动成功
ss -tlnp|grep n9e-server
```

如果启动不起来，或者进程启动了，但是端口没有在监听，都有问题，需要查阅日志，日志在server.log里以及logs目录里。上面是用nohup启动，更好的方式是用systemd来托管，service文件在`etc/service`下，至于如何使用systemd来管理，比较简单，这里就不赘述了，新手可以百度一下。

下面就可以用浏览器访问服务端的端口（在服务端的yml配置中可以看到http的监听端口）进入系统了，系统启动的时候会默认初始化一个`root`账号，密码是`root.2020` 登录之后没啥数据，毕竟我们还没有部署采集程序，下一小节开始部署客户端采集程序。

## 部署客户端

> 客户端的代码在这里：[https://github.com/n9e/n9e-agentd](https://github.com/n9e/n9e-agentd)

压缩包里其实默认打了客户端的二进制和相关配置，直接`nohup ./n9e-agentd -c etc/agentd.yml &> agentd.log &`就可以启动，如果启动失败，就查看日志，日志在agentd.log和logs目录下。

如上，就完成了整个单机版的部署，如果想多监控几台机器，只需要把客户端相关文件打个包，拷贝到目标机器上，修改agentd.yml中的服务端地址，即可启动验证。具体要把哪些文件打包呢？参考下面的命令：

```
tar zcvf n9e-agentd.tar.gz n9e-agentd etc/agentd.yml etc/conf.d etc/service/n9e-agentd.service
```

OK，把n9e-agentd.tar.gz分发到你要监控的机器上，解包之后修改agentd.yml中的服务端连接地址（搜索endpoint关键字），即可启动测试。
