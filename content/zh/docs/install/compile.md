
---
title: "通过源码编译安装"
linkTitle: "源码编译"
date: 2020-02-25
description: >
  如果您熟悉golang，了解基本的源码编译知识，可以通过这种方式来编译安装，Nightingale所有后端组件都是golang编写，不再有python模块，极致的简单
---

本节假设您已经了解golang编译基本常识，了解GOPATH环境变量代表的意思，建议使用golang1.12以上的版本。系统依赖mysql、redis，假设您已经安装完毕，并配置了mysql和redis的访问密码。本节所述为单机安装方式。

如果是新手，建议直接看演示视频，跟着视频来一步一步操作：[源码安装演示](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-install-src.mp4)

## 1、源码编译

```bash
# 该项目没有使用go module管理，需要放到github.com/didi下编译
mkdir -p $GOPATH/src/github.com/didi
cd $GOPATH/src/github.com/didi

# clone代码并编译打包，pack时会自动build，打包成一个tar.gz
git clone https://github.com/didi/nightingale.git
cd nightingale && ./control build && ./control pack
```

如果懒得编译，也可以参看《[镜像快速体验](../didiyun/)》一节去创建云主机，在云主机的`/home/n9e`目录下也可以拿到一个打包好的压缩包，名字叫n9e-xx.tar.gz，只要是64位Linux可以直接复用发布包

## 2、初始化数据库

sql文件已为您准备好，解开发布包，在sql目录，直接导入mysql即可：

```bash
mysql -uroot -p < sql/n9e_hbs.sql
mysql -uroot -p < sql/n9e_mon.sql
mysql -uroot -p < sql/n9e_uic.sql
```

## 3、修改配置文件

配置文件在etc目录，着重看一下mysql.yml，修改mysql访问的用户名和密码，另外redis密码默认为空，如果您配置了redis的访问密码，需要对应的修改monapi和judge的配置文件，将redis密码配置好。另外在`etc/address.yml`下可以看到各个模块监听的端口，如果与本地其他服务端口冲突了，就需要手工修改一下啦。

## 4、启动各模块进程

发布包里默认提供了一个control脚本，用来启停服务，直接执行`./control start all`即可启动所有模块，`./control status`可以查看各模块进程是否都已启动，夜莺共有6个核心模块，注意一下进程数是否正确。最后安装一下nginx，nginx有个示例配置文件在etc/nginx.conf，注意修改pub目录指向真实路径。至此，单机版就部署成功了。访问nginx即可看到页面。如果发现nginx日志里出现权限报错，检查机器selinux配置，尝试关闭selinux解决：

```bash
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

## 5、交代一些额外信息

如果在全部对象页面看不到机器，或者看到了机器但是看不到监控数据，请检查所有的logs目录下的文件，一定可以找到线索。比如是不是ifconfig、route等命令缺失？少了net-tools库？机器环境的问题还请baidu解决哈~

Nightingale本身的核心模块不提供告警发送功能，只是把告警消息推送到redis里就算完事，因为不同公司的告警通道各异，有的希望用邮件接收、有的希望用短信，或者是电话、微信、钉钉、自研IM等，这块希望各公司自行适配，共建社区。邮件的发送有SMTP标准，所以初期可以使用邮件测试，这里提供一个邮件的发送模块：[mail-sender](https://github.com/n9e/mail-sender) 作为样例。

