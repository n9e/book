
---
title: "利用滴滴云镜像快速体验"
linkTitle: "镜像安装"
date: 2020-02-26
description: >
  这是最快的方式，我们把夜莺做成了滴滴云OS镜像，直接基于这个镜像创建虚拟机即可体验，全过程5分钟内搞定，创建虚拟机的时候可以选择按量付费，便宜的很
---


## 1、[注册滴滴云领取红包](https://i.didiyun.com/27dt3TiddGd)

如果已经有账号可跳过此步骤

## 2、[创建云服务器DC2](https://app.didiyun.com/#/dc2/add)

*有了账号我们开始创建云主机，使用夜莺的镜像一键安装，如果对云资源操作不熟悉，可以 [点击查看演示视频](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-install-didiyun.mp4)*

付费方式选择“按时长”，会便宜一些，各配置不再一一赘述。注意在**镜像**这里，选择“一键部署镜像”，然后选择Nightingale的镜像即可，如图所示：

![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-didiyun-image-choose.png)

## 3、访问平台并修改密码

待虚机起来之后，镜像里边内置的初始化脚本会自动安装夜莺各个模块组件，大约等待2分钟，即可安装完毕，然后浏览器里输入该虚机的eip，即可访问到平台页面，使用root账号登陆，密码也是root，不要勾选那个LDAP的选项。登陆成功之后立马点击平台右上角的个人设置，修改root密码，即刻体验吧 :-)

## 4、交代一些额外信息

镜像里自带mysql、nginx、redis，nginx监听在80，如果访问不通，需要检查安全组规则。夜莺部署在`/home/n9e`目录，配置文件在`/home/n9e/etc`目录，可以看到mysql.yml，里边可以找到mysql的root密码

