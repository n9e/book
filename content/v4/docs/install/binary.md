
---
title: "二进制方式部署"
linkTitle: "二进制方式部署"
date: 2020-02-24
description: >
  一步一步手把手教学，讲解如何搭建夜莺服务端各个组件，当前是v4版本，v5预计还需要1~2个月，新用户建议等v5
---

## 视频教程

[手把手细致教学视频](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&__biz=MzU3ODAxNTIzMQ==&scene=1&album_id=1858590667128553475&count=3#wechat_redirect)

## 搭建文档

- [快速学习搭建单机版本](https://gocn.vip/topics/11958)

单机版本默认使用单机的m3db作为存储，m3db一定要和夜莺部署到一台机器上，且不能使用pve的虚拟机。也可以不用m3db做存储，使用 [n9e-tsdb](https://github.com/n9e/n9e-tsdb)。如果要上生产，m3db建议使用集群版本，集群版本的搭建方式建议参照 [M3官网](https://m3db.io/) 也可以参照[这个文档](https://n9e.didiyun.com/docs/install/m3db/)学习搭建m3集群

## 开始使用

登录 web，账号 `root`，密码 `root.2020`，登入后第一步先要修改密码，如果 `nginx` 报权限类的错误，检查 `selinux` 是否关闭以及防火墙策略是否异常，如下命令可关闭 `selinux` 和防火墙。

```shell script
setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disabled#' /etc/selinux/config
systemctl disable firewalld.service
systemctl stop firewalld.service
```

