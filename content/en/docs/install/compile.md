
---
title: "Compile and install through source code"
linkTitle: "Source code compilation"
date: 2020-02-25
description: >
  If you are familiar with golang and understand the basic source code compilation knowledge, you can compile and install in this way.All back-end components of Nightingale are written in golang and no longer have python modules.
---

This section assumes that you already know the basics of golang compilation and understand the meaning of the GOPATH environment variable. It is recommended to use golang 1.12 or above. The system depends on mysql and redis. We assume that you have installed and configured the access password for mysql and redis. This section describes the stand-alone installation method.

If you are a novice, it is recommended to watch the demo video directly and follow the video to operate：[Source code installation demo](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-install-src.mp4)

## 1、Source code compilation

```bash
# This project is not managed by go module, and needs to be compiled under github.com/didi
mkdir -p $GOPATH/src/github.com/didi
cd $GOPATH/src/github.com/didi

# Clone the code and compile and package it. It will be automatically built during packing and packaged into a tar.gz
git clone https://github.com/didi/nightingale.git
cd nightingale && ./control build && ./control pack
```

If you are too lazy to compile, you can also refer to《[Quick mirror experience](../didiyun/)》to create a cloud host.A packaged compressed package named n9e-xx.tar.gz can be obtained in the cloud host's `/home /n9e` directory, and the distribution package can be directly reused on 64-bit Linux.

## 2、Initialize the database

The sql file is ready for you, unpack the development package, and directly import mysql in the sql directory:

```bash
mysql -uroot -p < sql/n9e_hbs.sql
mysql -uroot -p < sql/n9e_mon.sql
mysql -uroot -p < sql/n9e_uic.sql
```

## 3、Modify configuration file

When the configuration file is in the etc directory, you need to take a look at mysql.yml and modify the user name and password for mysql access. The redis password is empty by default. If you configure a redis access password, you need to modify the configuration files of monapi and judge accordingly to configure the redis password. In addition, under `etc /address.yml`, you can see the ports that each module listens to. If it conflicts with other local service ports, you need to modify it manually.

## 4、Start each module process

There's a control script in tarball for start/stop processes. `./control start all` to start all processes. `./control status` to check all processes. Nightingale has 6 core modules. Please pay attention to whether the number of processes is correct. Finally, install nginx. Nginx has a sample configuration file in `etc/nginx.conf`, modify pub directory to real path. At this point, the stand-alone version is deployed successfully. Visit nginx to see the page. If the permission error is found in the nginx log, check the SELinux configuration of the machine and try to close SELinux.

```bash
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

## 5、Some extra information

If you cannot see the machine on all object pages, or if you see the machine but cannot see the monitoring data, please check all the files in the logs directory. For example, are commands such as ifconfig and route missing? Missing net-tools library? If you encounter problems with the machine's initial environment, please google to solve it.

The core module of Nightingale itself does not provide the alarm sending function, but just push the alarm message to redis and it is over. Because the alarm channels of different companies are different, some hope to receive by mail, some hope to use SMS, or phone, WeChat, Dingding, self-developed IM, etc. This hopes that each company will adapt itself and build a community. There are SMTP standards for sending emails, so you can use email testing at the beginning. Here is a mail sending module: [mail-sender] (https://github.com/n9e/mail-sender) as an example.

