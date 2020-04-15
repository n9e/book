---
title: "通过rpm包安装"
linkTitle: "RPM安装"
date: 2020-4-04
description: >
  使用rpm包方式，安装n9e各个组件
---

n9e 各个组件已打包为 rpm 包,可根据具体需求下载安装

## 更新

2020.4.15 更新内容

1. n9e 版本升级到 1.3.0-438ec4a
2. 修复目录权限问题导致 systemd 启动组件失败

## rpm 包介绍

1.3.0 版本

| 包名                                        | 依赖包    | 介绍                 | 安装路径           |
| :------------------------------------------ | :-------- | :------------------- | :----------------- |
| n9e-web-1.3.0-438ec4a.el7.x86_64.rpm        | 无        | n9e web 前端静态文件 | /usr/local/n9e/pub |
| n9e-sql-1.3.0-438ec4a.el7.x86_64.rpm        | 无        | n9e sql 文件         | /usr/local/n9e/sql |
| n9e-nginx-conf-1.3.0-438ec4a.el7.x86_64.rpm | 无        | n9e nginx 配置文件   | /usr/local/n9e/etc |
| n9e-collector-1.3.0-438ec4a.el7.x86_64.rpm  | net-tools | n9e collector 组件   | /usr/local/n9e     |
| n9e-tsdb-1.3.0-438ec4a.el7.x86_64.rpm       | 无        | n9e tsdb 组件        | /usr/local/n9e     |
| n9e-transfer-1.3.0-438ec4a.el7.x86_64.rpm   | 无        | n9e transfer 组件    | /usr/local/n9e     |
| n9e-monapi-1.3.0-438ec4a.el7.x86_64.rpm     | 无        | n9e monapi 组件      | /usr/local/n9e     |
| n9e-judge-1.3.0-438ec4a.el7.x86_64.rpm      | net-tools | n9e judge 组件       | /usr/local/n9e     |
| n9e-index-1.3.0-438ec4a.el7.x86_64.rpm      | net-tools | n9e index 组件       | /usr/local/n9e     |

rpm 包下载地址

[https://dl.cactifans.com/n9e/1.3.0/](https://dl.cactifans.com/n9e/1.3.0/)

sha1

```
98b9663e8ea81b925582638027873d9bb3dacfaf  n9e-collector-1.3.0-438ec4a.el7.x86_64.rpm
b8d3847471065f635b50e1f66aac4f4a08e5beb1  n9e-index-1.3.0-438ec4a.el7.x86_64.rpm
144316d0505b85c5703e7381d1f9eb42c81cfa2c  n9e-judge-1.3.0-438ec4a.el7.x86_64.rpm
9d1eabae946fce034c9ef1ebb23f5b51871b43ec  n9e-monapi-1.3.0-438ec4a.el7.x86_64.rpm
892116d0a065a8ca8ca69779b69e4f70b34842ae  n9e-nginx-conf-1.3.0-438ec4a.el7.x86_64.rpm
572c77d7b61e8f374f393ae4bd6e708e1f87f692  n9e-sql-1.3.0-438ec4a.el7.x86_64.rpm
86f2ec4a6a49954b1a370eba5d6e22f9d83e692f  n9e-transfer-1.3.0-438ec4a.el7.x86_64.rpm
f8e33af712dad5004afdb5606753a8c396fffe89  n9e-tsdb-1.3.0-438ec4a.el7.x86_64.rpm
602b9ac5b06ea7d8315ac9042492507083bfca74  n9e-web-1.3.0-438ec4a.el7.x86_64.rpm
dbdc842f68bfe1b152ab47fecc85373341690183  n9e-1.3.0-438ec4a.el7.x86_64.rpm-bundle.tar.gz
```

## 说明

- 除 collector 组件使用 root 用户启动外，其他组件使用 n9e 用户启动
- 日志统一在/usr/local/n9e/模块名称/
- 按需安装对应模块即可

### 安装使用

## All in one 安装

将所有组件安装在一台机器上，下载 n9e-1.3.0-438ec4a.el7.x86_64.rpm-bundle.tar.gz

```
tar zxvf n9e-1.3.0-438ec4a.el7.x86_64.rpm-bundle.tar.gz
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
systemctl enable --now n9e-collector n9e-tsdb n9e-transfer n9e-monapi n9e-judge n9e-index
```

使用浏览器打开http://ip 即可访问，默认账号 root 密码 root

## 分布式安装

### Collector 安装(采集客户端安装)

collector 为 n9e 采集客户端,需要安装在客户端

```bash
yum install n9e-collector-1.3.0-438ec4a.el7.x86_64.rpm  -y
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
yum install n9e-web-1.3.0-438ec4a.el7.x86_64.rpm n9e-nginx-conf-1.3.0-438ec4a.el7.x86_64.rpm -y
```

安装后静态文件路径为/usr/local/n9e/pub，同时提供一个 nginx 配置文件/usr/local/n9e/etc/nginx.conf,建议安装好 nginx 之后直接替换即可，如组件为分布式部署，建议根据实际配置修改 nginx 配置文件里对应的 upstream 服务器地址。

### 表结构安装

n9e 需要使用 mysql 数据库,安装好 mysql 数据库之后，使用

```
yum install n9e-sql-1.3.0-438ec4a.el7.x86_64.rpm -y
```

可在/usr/local/n9e/sql 目录看到 3 个 sql 文件，导入数据库即可。同时注意修改组件/usr/local/n9e/etc/mysql.yml 文件里的数据库配置信息。

### 其他组件安装

其余组件直接使用 yum 安装,注意配置/usr/local/n9e/etc 下的配置文件信息
