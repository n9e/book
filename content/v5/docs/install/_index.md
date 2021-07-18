
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
如果启动不起来，或者进程启动了，但是端口没有在监听，都有问题，需要查阅日志，执行 `journalctl -u n9e-server -f` 查看 n9e-server 的启动日志，以及查看 /opt/n9e/server/logs 下面的日志

安装成功之后，就可以用浏览器访问服务端的端口（在服务端的yml配置中可以看到http的监听端口，默认是8000）进入系统了，系统启动的时候会默认初始化一个`root`账号，密码是`root.2020` 登录之后没啥数据，毕竟我们还没有部署采集程序，下一小节开始部署客户端采集程序。

## 部署客户端

> 客户端的代码在这里：[https://github.com/n9e/n9e-agentd](https://github.com/n9e/n9e-agentd)

```bash
mkdir -p /opt/n9e
cd /opt/n9e
wget 116.85.64.82/n9e-agentd.tar.gz
tar zxvf n9e-agentd.tar.gz
cd /opt/n9e/agentd
nohup ./bin/n9e-agentd -c etc/agentd.yaml &> agentd.log &

# 通过下面命令可以n9e-agentd的进程，如果进程存在，说明启动成功了，如果启动失败，就查看日志，日志在agentd.log下
ps -ef|grep n9e-agentd|grep -v grep
```

如上，就完成了整个单机版的部署，如果想多监控几台机器，只需要修改/opt/n9e/agentd/etc/agentd.yaml中的服务端连接地址（搜索endpoint关键字），然后重启打下包，把压缩包拷贝到目标机器上，即可启动验证。如果想要systemd托管的话，n9e-agentd.tar.gz解压后的systemd文件下

```
#重新打包
cd /opt/n9e/
tar czvf n9e-agentd.tar.gz agentd
```