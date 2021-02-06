
---
title: "二进制方式部署"
linkTitle: "二进制方式部署"
date: 2020-02-24
description: >
  一步一步手把手教学，讲解如何搭建夜莺服务端各个组件，当前是v3版本，模块较多，等v4的时候，会合并服务端组件，敬请期待
---

## setup01

准备一台系统为 CentOS7.X 的虚拟机或物理机，并安装完成 `mysql`、`redis`、`nginx`软件，简单 `yum` 安装即可，生产环境可寻求运维或DBA同学帮忙部署。

```shell script
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum install -y mariadb* redis nginx
```

## setup02

下载我们已经编译好的二进制包到 `/home/n9e` 目录，如果想要更换安装目录，需要修改 `nginx.conf`，建议先用这个目录，熟悉之后，在进行自定义。

```shell script
mkdir -p /home/n9e
cd /home/n9e
wget http://116.85.64.82/n9e.tar.gz
tar zxvf n9e.tar.gz
```

## setup03

初始化数据库，这里假设使用 `root` 账号，密码为 `1234`，如果不是这个账号密码，需要修改 `/home/n9e/etc/mysql.yml`。

```shell script
cd /home/n9e/sql
mysql -uroot -p1234 < n9e_ams.sql
mysql -uroot -p1234 < n9e_hbs.sql
mysql -uroot -p1234 < n9e_job.sql
mysql -uroot -p1234 < n9e_mon.sql
mysql -uroot -p1234 < n9e_rdb.sql
```

## setup04

redis 配置修改，默认配置的 `6379` 端口，密码为空，如果默认配置不对，可以执行如下命令，看到多个配置文件里有 redis 相关配置，挨个检查并修改即可。

```shell script
cd /home/n9e/etc
grep redis -r .
```

## setup05

下载前端静态资源文件，放到默认的 `/home/n9e` 目录下，如果要更改目录，需要修改后面提到的 `nginx.conf` 文件。

前端的源码单独拆了一个 repo，地址是： https://github.com/n9e/fe 没有和 nightingale 放在一起。

```shell script
cd /home/n9e
wget http://116.85.64.82/pub.tar.gz
tar zxvf pub.tar.gz
```

## setup06

覆盖 `nginx.conf`，建议大家还是看一下这个配置，熟悉一下 nginx 的配置，夜莺不同 web 侧组件就是通过 nginx 的不同 `location` 区分的，覆盖完了配置记得 `reload` 一下或者重启 nginx。

```shell script
cp etc/nginx.conf /etc/nginx/nginx.conf
systemctl restart nginx
```

## setup07

检查 `identity.yml`，要保证这个 shell 可以正常获取本机 ip，如果实在不能正常获取，自己又不懂 shell，那么在 `specify` 字段写固定也可以。

```yaml
# 用来做心跳，给服务端上报本机ip
ip:
  specify: ""
  shell: ifconfig `route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|head -n 1

# MON、JOB的客户端拿来做本机标识
ident:
  specify: ""
  shell: ifconfig `route|grep '^default'|awk '{print $NF}'`|grep inet|awk '{print $2}'|head -n 1
```

## setup08

检查 `agent.yml` 的几个 shell，挨个检查是否可以跑通，跑不通就改成适合自己环境的，实在不会改，直接写固定，比如 `disk` 部分，如果固定为 `80Gi` ，那么就可以写为：`disk: echo 80Gi`。

```yaml
report:
  # ...
  sn: dmidecode -s system-serial-number | tail -n 1

  fields:
    cpu: cat /proc/cpuinfo | grep processor | wc -l
    mem: cat /proc/meminfo | grep MemTotal | awk '{printf "%dGi", $2/1024/1024}'
    disk: df -m | grep '/dev/' | grep -v '/var/lib' | grep -v tmpfs | awk '{sum += $2};END{printf "%dGi", sum/1024}'
```

## setup09

启动各个进程，包括 `mysql`、`redis`、`nginx`，夜莺的各个组件直接用 `control` 脚本启动即可，后续上生产环境，可以用`systemd` 之类的进行托管。

```shell script
cd /home/n9e
./control start all
```

## setup10

登录 web，账号 `root`，密码 `root.2020`，登入后第一步先要修改密码，如果 `nginx` 报权限类的错误，检查 `selinux` 是否关闭以及防火墙策略是否异常，如下命令可关闭 `selinux` 和防火墙。

```shell script
setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disabled#' /etc/selinux/config
systemctl disable firewalld.service
systemctl stop firewalld.service
systemctl stop NetworkManager
systemctl disable NetworkManager
```

## setup11

如果前10步骤都部署完成但仍然没有搭建起来，你可能需要[查看视频教程](https://mp.weixin.qq.com/s/OAEQ-ec-QM74U0SGoVCXkg)。
