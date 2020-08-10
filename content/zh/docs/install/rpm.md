---
title: "通过rpm包安装"
linkTitle: "RPM安装"
date: 2020-4-04
description: >
  使用rpm包方式，安装n9e各个组件
---

n9e 各个组件已打包为 rpm 包,可根据具体需求下载安装

## 更新

2020.7.5 更新内容
1.n9e 版本升级到 2.7.2-dbd81ee

## rpm 包介绍

2.7.2 版本

| 包名                                        | 依赖包    | 介绍                 | 安装路径           |
| :------------------------------------------ | :-------- | :------------------- | :----------------- |
| n9e-web-2.7.2-dbd81ee.el7.x86_64.rpm        | 无        | n9e web 前端静态文件 | /usr/local/n9e/pub |
| n9e-sql-2.7.2-dbd81ee.el7.x86_64.rpm        | 无        | n9e sql 文件         | /usr/local/n9e/sql |
| n9e-nginx-conf-2.7.2-dbd81ee.el7.x86_64.rpm | 无        | n9e nginx 配置文件   | /usr/local/n9e/etc |
| n9e-collector-2.7.2-dbd81ee.el7.x86_64.rpm  | net-tools | n9e collector 组件   | /usr/local/n9e     |
| n9e-tsdb-2.7.2-dbd81ee.el7.x86_64.rpm       | 无        | n9e tsdb 组件        | /usr/local/n9e     |
| n9e-transfer-2.7.2-dbd81ee.el7.x86_64.rpm   | 无        | n9e transfer 组件    | /usr/local/n9e     |
| n9e-monapi-2.7.2-dbd81ee.el7.x86_64.rpm     | 无        | n9e monapi 组件      | /usr/local/n9e     |
| n9e-judge-2.7.2-dbd81ee.el7.x86_64.rpm      | net-tools | n9e judge 组件       | /usr/local/n9e     |
| n9e-index-2.7.2-dbd81ee.el7.x86_64.rpm      | net-tools | n9e index 组件       | /usr/local/n9e     |

2.7.2 版本 rpm 包下载地址
[https://dl.cactifans.com/n9e/2.7.2/](https://dl.cactifans.com/n9e/2.7.2/)
1.3.0 版本 rpm 包下载地址
[https://dl.cactifans.com/n9e/1.3.0/](https://dl.cactifans.com/n9e/1.3.0/)

sha1

```
060428277ade7f31f89b483a31061f40f1f6d634  n9e-collector-2.7.2-dbd81ee.el7.x86_64.rpm
1871e501e9cb4414719c4e82307dde14e7c5b419  n9e-index-2.7.2-dbd81ee.el7.x86_64.rpm
51f68e28fe746a0d271603bf89239e6f900020c8  n9e-judge-2.7.2-dbd81ee.el7.x86_64.rpm
a8a428e68f56861fd00572a309f02677c040e0ea  n9e-monapi-2.7.2-dbd81ee.el7.x86_64.rpm
155798a6bfc766a13688f840d56274a2c6edab5c  n9e-nginx-conf-2.7.2-dbd81ee.el7.x86_64.rpm
4329fb0fb16c05790681d5d0cea29fc1e21f4214  n9e-sql-2.7.2-dbd81ee.el7.x86_64.rpm
b01919f2e4d9d4006ddc67e98a1c6377497c577d  n9e-transfer-2.7.2-dbd81ee.el7.x86_64.rpm
5c802c584e0960e8fb927736c4a1172537888975  n9e-tsdb-2.7.2-dbd81ee.el7.x86_64.rpm
578e2b013c77918f8d13c03cce1916e90f4172fe  n9e-web-2.7.2-dbd81ee.el7.x86_64.rpm
76efc6d2cc8c488a5ac8edd3734841a1de1cac32  n9e-2.7.2-dbd81ee.el7.x86_64.rpm-bundle.tar.gz
```

## 说明

- 除 collector 组件使用 root 用户启动外，其他组件使用 n9e 用户启动
- 日志统一在/usr/local/n9e/模块名称/
- 按需安装对应模块即可

### 安装使用

## All in one 安装

将所有组件安装在一台机器上，下载 n9e-2.7.2-dbd81ee.el7.x86_64.rpm-bundle.tar.gz

```
tar zxvf n9e-2.7.2-dbd81ee.el7.x86_64.rpm-bundle.tar.gz
yum install n9e-* -y
```

安装之后，需要安装 nginx，mysql 组件。
mysql 导入表结构

```bash
mysql -uroot -p </usr/local/n9e/sql/n9e_hbs.sql
mysql -uroot -p </usr/local/n9e/sql/n9e_mon.sql
mysql -uroot -p </usr/local/n9e/sql/n9e_uic.sql
```

安全考虑，建议为 n9e 独立建立 mysql 用户，在 mysql 里创建 n9e 用户并授权

```
mysql>create user n9e@localhost identified by 'n9epwd123';
mysql>grant all on n9e_hbs.* to n9e@localhost;
mysql>grant all on n9e_mon.* to n9e@localhost;
mysql>grant all on n9e_uic.* to n9e@localhost;
```

已建立 n9e 用户,密码为 n9epwd123
替换默认 nginx 配置文件,并重启 nginx 服务

```
cp /usr/local/n9e/etc/nginx.conf /etc/nginx/
systemctl restart nginx
```

启动所有组件

```bash
systemctl enable --now n9e-index n9e-tsdb n9e-transfer n9e-monapi n9e-judge n9e-collector
```

使用浏览器打开http://ip 即可访问，默认账号 root 密码 root

## 分布式安装

### Collector 安装(采集客户端安装)

collector 为 n9e 采集客户端,需要安装在客户端

```bash
yum install n9e-collector-2.7.2-dbd81ee.el7.x86_64.rpm  -y
```

即可完成安装。安装后修改配置文件/usr/local/n9e/etc/address.yml

```
...
monapi:
  http: 0.0.0.0:5800
  addresses:
    - 127.0.0.1

transfer:
  http: 0.0.0.0:5810
  rpc: 0.0.0.0:5811
  addresses:
    - 127.0.0.1
...
```

collector 需要与 monapi 与 transfer 通信，需要修改 127.0.0.1 地址为实际的组件地址，多个组件可写多行，配置文件为 yaml 格式，修改时注意格式。
修改后，使用以下命令启动

```
systemctl enable --now n9e-collector
```

### 前端安装

n9e 前端为静态文件，建议使用 ningx 进行配置

```bash
yum install n9e-web-2.7.2-dbd81ee.el7.x86_64.rpm n9e-nginx-conf-2.7.2-dbd81ee.el7.x86_64.rpm -y
```

安装后静态文件路径为/usr/local/n9e/pub，同时提供一个 nginx 配置文件/usr/local/n9e/etc/nginx.conf,建议安装好 nginx 之后直接替换即可，如组件为分布式部署，建议根据实际配置修改 nginx 配置文件里对应的 upstream 服务器地址。

### 表结构安装

n9e 需要使用 mysql 数据库,安装好 mysql 数据库之后，使用

```
yum install n9e-sql-2.7.2-dbd81ee.el7.x86_64.rpm -y
```

可在/usr/local/n9e/sql 目录看到 3 个 sql 文件，导入数据库即可。同时注意修改组件/usr/local/n9e/etc/mysql.yml 文件里的数据库配置信息。

### 其他组件安装

其余组件直接使用 yum 安装,注意配置/usr/local/n9e/etc 下的配置文件信息
