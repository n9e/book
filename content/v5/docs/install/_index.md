
---
title: "安装部署"
linkTitle: "安装部署"
weight: 3
date: 2021-06-23
description: >
  阅读本章节，一定要先阅读《[夜莺简介](/docs/intro/)》，了解整体的架构，这样出了问题方能有思路排查。因为大部分生产环境都是使用CentOS7，后面的讲解我们都将基于此发行版展开，所有二进制都是X86架构下编译而成，这点请注意。
---

回顾《[夜莺简介](/docs/intro/)》中的架构讲解，整个数据流是`agentd->server<->storage`，即agentd采集监控数据，推送给server，server会做各种处理，然后交由存储来持久化数据，后续server要读取数据的时候就从存储来读。所以，我们的部署顺序就是从被依赖方开始部署：1.部署存储 2.部署服务端 3.部署客户端

## 部署存储

> TODO: 部署存储这个章节还需要补充一下

存储是可插拔的方式，简单起见，我们选择单机版Prometheus来快速开始，部署Prometheus的方式如下：

```bash
mkdir -p /opt/prom
cd /opt/prom
wget xxx
tar zxf xxx
cd xxx
sh install.sh
```

## 部署服务端

服务端依赖mysql数据库，请自行安装，v5版本的数据库表结构和之前的版本不兼容，所以没法复用之前版本的数据库，这点请注意。

#### 1. 下载安装包

```bash
mkdir -p /opt/n9e
cd /opt/n9e
wget xxx
tar zxf xxx
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
./n9e-server
```

上面的命令是启动了一个前台进程，如果起不来，程序崩溃，相关错误就会在命令行直接打出来，比较容易排错，如果命令行里的错误无法帮助定位问题，请查阅logs目录下的相关日志。如果前台进程启动发现一切正常，`Ctrl+C`停掉进程，重新通过nohup启动，扔到后台来运行：`nohup ./n9e-server &> server.log &` 当然，更好的方式是用systemd来托管，service文件可以在`etc/service`下找到。

下面就可以用浏览器访问服务端的端口（在服务端的yml配置中可以看到http的监听端口）进入系统了，系统启动的时候会默认初始化一个`root`账号，密码是`root.2020` 登录之后什么数据都没有，毕竟我们还没有部署采集程序，下一小节开始部署客户端采集程序。

## 部署客户端

压缩包里其实默认打了客户端的二进制和相关配置，直接`./n9e-agentd`即可在前台启动客户端做验证，如果一切正常，应该可以在WEB上看到本机上报的监控数据，资源管理页面里也可以看到这个资源信息。验证没问题就可以`Ctrl+C`停掉进程，然后用nohup正经启动了：`nohup ./n9e-agentd &> agentd.log &` 当然，更好的方式还是用systemd来托管，service文件可以在`etc/service`下找到。

如上，就完成了整个单机版的部署，如果想多监控几台机器，只需要把客户端相关文件打个包，拷贝到目标机器上，修改agentd.yml中的服务端地址，即可启动验证。具体要把哪些文件打包呢？参考下面的命令：

```
tar zcvf n9e-agentd.tar.gz n9e-agentd etc/agentd.yml service/n9e-agentd.service
```

OK，把n9e-agentd.tar.gz分发到你要监控的机器上，解包之后修改agentd.yml中的服务端连接地址（搜索endpoint关键字），即可启动测试。


