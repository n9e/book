---
title: "Install via rpm package"
linkTitle: "RPM installation"
date: 2020-4-13
description: >
  Use rpm package to install n9e components
---

The n9e components have been packaged as rpm packages, which can be downloaded and installed as needed

## rpm package introduction

| Package names                                   | Dependent packages    | Introduction         | installation path           |
| :------------------------------------------- | :-------- | :------------------- | :----------------- |
| n9e-web-1.2.0-g59f2bb0.el7.x86_64.rpm        | no        | n9e web Front-end static files | /usr/local/n9e/pub |
| n9e-sql-1.2.0-g59f2bb0.el7.x86_64.rpm        | no       | n9e sql file        | /usr/local/n9e/sql |
| n9e-nginx-conf-1.2.0-g59f2bb0.el7.x86_64.rpm | no        | n9e nginx conf   | /usr/local/n9e/etc |
| n9e-collector-1.2.0-g59f2bb0.el7.x86_64.rpm  | net-tools | n9e collector Components   | /usr/local/n9e     |
| n9e-tsdb-1.2.0-g59f2bb0.el7.x86_64.rpm       | no        | n9e tsdb Components        | /usr/local/n9e     |
| n9e-transfer-1.2.0-g59f2bb0.el7.x86_64.rpm   | no        | n9e transfer Components    | /usr/local/n9e     |
| n9e-monapi-1.2.0-g59f2bb0.el7.x86_64.rpm     | no        | n9e monapi Components      | /usr/local/n9e     |
| n9e-judge-1.2.0-g59f2bb0.el7.x86_64.rpm      | net-tools | n9e judge Components       | /usr/local/n9e     |
| n9e-index-1.2.0-g59f2bb0.el7.x86_64.rpm      | net-tools | n9e index Components       | /usr/local/n9e     |

rpm package download address

[https://dl.cactifans.com/n9e/](https://dl.cactifans.com/n9e/)

sha1

```
43ad9fac28e59a58644a3381d966398a9e397543  n9e-web-1.2.0-g59f2bb0.el7.x86_64.rpm
754d2be782088817a0cef8fd23b274248d2f0da2  n9e-sql-1.2.0-g59f2bb0.el7.x86_64.rpm
126ffc6beb8ac5c175d9ca086f4cf21222362623  n9e-nginx-conf-1.2.0-g59f2bb0.el7.x86_64.rpm
105eb15870694cb9bd381d834e3afd19c9da8fff  n9e-collector-1.2.0-g59f2bb0.el7.x86_64.rpm
0c504c09d625fd34384ad0065c0aaeac73a04587  n9e-tsdb-1.2.0-g59f2bb0.el7.x86_64.rpm
8b94ca07621c06bacecbebaa2adedc9c4220c790  n9e-transfer-1.2.0-g59f2bb0.el7.x86_64.rpm
b1c4f9648606b3bf6d9aa2be3156feb26cbd183b  n9e-monapi-1.2.0-g59f2bb0.el7.x86_64.rpm
e67638dd0e9052853d43b79af9bcf802b2a8b675  n9e-judge-1.2.0-g59f2bb0.el7.x86_64.rpm
6bd075f4d1dd1a260e3c2491a324b5c7c53dc5fc  n9e-index-1.2.0-g59f2bb0.el7.x86_64.rpm
60a035b25a509a92fec2746a3983e9b52aa39548  n9e-1.2.0-g59f2bb0.el7.x86_64.rpm-bundle.tar.gz
```

## Explanation

- Except that the collector component is started by the root user, other components are started by the n9e user
- The log is in the path: /usr /local /n9e /module name /
- Install the corresponding modules as needed

### Install and use

## All in one installation

Install all components on one machine

```
yum install n9e-web-1.2.0-g59f2bb0.el7.x86_64.rpm n9e-sql-1.2.0-g59f2bb0.el7.x86_64.rpm n9e-nginx-conf-1.2.0-g59f2bb0.el7.x86_64.rpm n9e-collector-1.2.0-g59f2bb0.el7.x86_64.rpm n9e-tsdb-1.2.0-g59f2bb0.el7.x86_64.rpm n9e-transfer-1.2.0-g59f2bb0.el7.x86_64.rpm n9e-monapi-1.2.0-g59f2bb0.el7.x86_64.rpm n9e-judge-1.2.0-g59f2bb0.el7.x86_64.rpm n9e-index-1.2.0-g59f2bb0.el7.x86_64.rpm
```

After installation, you need to install nginx, mysql components.
mysql import table structure

```bash
mysql -uroot -p </usr/local/n9e/sql/n9e_hbs.sql
mysql -uroot -p </usr/local/n9e/sql/n9e_mon.sql
mysql -uroot -p </usr/local/n9e/sql/n9e_uic.sql
```

For security reasons, it is recommended to establish a mysql user for n9e independently, create and authorize n9e user in mysql (if using MySQL8, you need to use mysql_native_password authentication plugin)

```
mysql>create user n9e@localhost identified by 'n9epwd123';
mysql>grant all on n9e_hbs.* to n9e@localhost;
mysql>grant all on n9e_mon.* to n9e@localhost;
mysql>grant all on n9e_uic.* to n9e@localhost;
```

An n9e user has been created and the password is n9epwd123
Modify /usr/local/n9e/etc/mysql.yml

```
 ---
uic:
  addr: "n9e:n9epwd123@tcp(127.0.0.1:3306)/n9e_uic?charset=utf8&parseTime=True&loc=Asia%2FShanghai"
  max: 16
  idle: 4
  debug: false
mon:
  addr: "n9e:n9epwd123@tcp(127.0.0.1:3306)/n9e_mon?charset=utf8&parseTime=True&loc=Asia%2FShanghai"
  max: 16
  idle: 4
  debug: false
hbs:
  addr: "n9e:n9epwd123@tcp(127.0.0.1:3306)/n9e_hbs?charset=utf8&parseTime=True&loc=Asia%2FShanghai"
  max: 16
  idle: 4
  debug: false
```

Replace nginx configuration and restart nginx

```
cp /usr/local/n9e/etc/nginx.conf /etc/nginx/
systemctl restart nginx
```

Start all components

```bash
systemctl enable --now n9e-collector n9e-tsdb n9e-transfer n9e-monapi n9e-judge n9e-index
```

Use a browser to open http: //ip to access, the default account root password root

## Distributed installation

### Collector installation (collection client installation)

collector is the n9e collection client, which needs to be installed on the client

```bash
yum install n9e-collector-1.2.0-g59f2bb0.el7.x86_64.rpm  -y
```

finish installation. Modify configuration file /usr/local/n9e/etc/address.yml after installation

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

The collector needs to communicate with monapi and transfer, so the 127.0.0.1 address needs to be modified to the actual component address. Multiple components can write multiple lines, the configuration file is in yaml format, pay attention to the format when modifying.
After modification, use the following command to start

```
systemctl enable --now n9e-collector
```

### Front-end installation

The front end of n9e is a static file, it is recommended to use ningx for configuration

```bash
yum install n9e-web-1.2.0-g59f2bb0.el7.x86_64.rpm n9e-nginx-conf-1.2.0-g59f2bb0.el7.x86_64.rpm -y
```

After installation, the static file path is /usr /local /n9e /pub, while providing an nginx configuration file /usr/local/n9e/etc/nginx.conf. It is recommended to replace it directly after installing nginx. If the component is a distributed deployment, it is recommended to modify the corresponding upstream server address in the nginx configuration file according to the actual configuration.

### Table structure installation

n9e depends on mysql database
```
yum install n9e-sql-1.2.0-g59f2bb0.el7.x86_64.rpm -y
```

You can see 3 sql files in the /usr /local /n9e /sql directory, and import them into the database. At the same time, pay attention to modify the database configuration information in the component /usr/local/n9e/etc/mysql.yml file.

### Installation of other components

The rest of the components are installed directly using yum, pay attention to the configuration file information under /usr /local /n9e /etc
