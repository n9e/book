
---
title: "安装部署"
linkTitle: "安装部署"
weight: 3
date: 2021-06-23
description: >
  阅读本章节，一定要先阅读《[夜莺介绍](/docs/intro/)》，了解整体的架构，这样出了问题方能有思路排查。因为大部分生产环境都是使用CentOS7，后面的讲解我们都将基于此发行版展开，所有二进制都是X86架构下编译而成，这点请注意。
---

> 演示环境：[http://116.85.46.86/](http://116.85.46.86/) 用户名：demo，密码：demo.2021

回顾《[夜莺介绍](/docs/intro/)》中的架构讲解，整个数据流是`agentd->server<->storage`，即agentd采集监控数据，推送给server，server会做各种处理，然后交由存储来持久化数据，后续server要读取数据的时候就从存储来读。所以，我们的部署顺序就是从被依赖方开始部署：1.部署服务端(包括存储) 2.部署客户端

## 部署服务端
我们提供了一个一键部署的命令，如果操作系统是centos的话，可以执行下面命令一键安装
```bash
# 需要以root权限执行，机器需要可以连接互联网
# 安装脚本做了3件事情
# 1. 安装promethues作为存储，夜莺支持对接多种存储，我们选择单机版Prometheus来快速开始
# 2. 安装mysql，root默认密码为1234
# 3. 安装n9e-server
curl -s http://116.85.64.82/install_n9e_server.sh|bash

# 进程如果启动了，理论上会监听2个端口，一个http端口一个rpc端口
# 通过下面命令可以查看端口是否在监听，如果端口都在监听，就说明启动成功
ss -tlnp|grep n9e-server
```

安装脚本的详细内容如下，如果机器的操作系统不是centos，可以根据自己的需求来做调整
<details>
<summary>点击查看安装脚本内容</summary>

```bash
#!/bin/bash

# 1.安装promethues作为存储，夜莺支持对接多种存储，我们选择单机版Prometheus来快速开始
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

# 2.安装mysql，root默认密码为1234
yum -y install mariadb*
# 假设机器的/home分区是个SSD的大分区，datadir设置为/home/mysql
# mkdir -p /home/mysql
# chown mysql:mysql /home/mysql
# sed -i '/^datadir/s/^.*$/datadir=\/home\/mysql/g' /etc/my.cnf
# 启动mysql进程
systemctl start mariadb.service
# 将mysql设置为开机自启动
systemctl enable mariadb.service
# 设置mysql root密码
mysql -e "SET PASSWORD FOR 'root'@'localhost' = PASSWORD('1234');"

# 3.安装n9e-server
mkdir -p /opt/n9e
cd /opt/n9e
wget 116.85.64.82/n9e-server-5.0.0-r3.tar.gz
tar zxvf n9e-server-5.0.0-r3.tar.gz
mysql -uroot -p1234 < /opt/n9e/server/sql/n9e.sql

cp /opt/n9e/server/etc/service/n9e-server.service /etc/systemd/system/
systemctl daemon-reload
systemctl enable n9e-server
systemctl restart n9e-server
systemctl status n9e-server
```

</details>


如果启动不起来，或者进程启动了，但是端口没有在监听，都有问题，需要查阅日志，执行 `journalctl -u n9e-server` 查看 n9e-server 的启动日志，以及查看 /opt/n9e/server/logs 下面的日志

安装成功之后，就可以用浏览器访问服务端的端口（在服务端的yml配置中可以看到http的监听端口，默认是8000）进入系统了，系统启动的时候会默认初始化一个`root`账号，密码是`root.2020` 登录之后没啥数据，毕竟我们还没有部署采集程序，下一小节开始部署客户端采集程序。

## 部署客户端

> 客户端的代码在这里：[https://github.com/n9e/n9e-agentd](https://github.com/n9e/n9e-agentd)

```bash
# 一键安装n9e-agentd
curl -s http://116.85.64.82/install_n9e_agentd.sh|bash

# 通过下面命令查看n9e-agentd的进程，如果进程存在，说明启动成功
# 如果启动失败，可通过 journalctl -u n9e-agentd 查看日志
ps -ef|grep n9e-agentd|grep -v grep
```

如上，就完成了整个单机版的部署，如果想多监控几台机器，只需要执行一键安装n9e-agentd的命令，安装完成之后
修改/opt/n9e/agentd/etc/agentd.yaml中的服务端连接地址（搜索endpoint关键字），然后重启n9e-agentd即可，重启命令 `systemctl restart n9e-agentd`

## 高可用集群部署

不同的时序数据库有不同的集群搭建方式，这里不展开讲解，我们主要讲解如何做夜莺组件的高可用。夜莺的服务端用到的核心组件就一个：n9e-server，n9e-server的集群部署也非常简单，就是找多个机器，部署多个n9e-server进程即可。

n9e-agentd主动连n9e-server推送数据，走的是http接口（即默认的8000端口，protobuf协议），建议的高可用方案是在多台n9e-server前面架设4层负载均衡，比如LVS或者haproxy，agentd连接vip，这样服务端扩缩容，都只需要变更vip和n9e-server的对应关系即可。如果不方便搞负载均衡，也可以把多个n9e-server的地址直接配置到agentd.yaml中，agentd会随机请求server，如果某一个连不上，会重试其他的server，这样做的问题，就是server如果扩缩容，就要修改所有的agentd的配置了。