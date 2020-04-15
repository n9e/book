
---
title: "Quick experience with Didi cloud mirroring"
linkTitle: "Mirror installation"
date: 2020-04-13
description: >
  This is the fastest way. We made the Nightingale a Didi cloud OS image. We can directly create a virtual machine based on this image, and the whole process will be done in 5 minutes. When creating a virtual machine, you can choose to pay by volume, which is very cheap.
---


## 1、[Register Didiyun to receive discount](https://i.didiyun.com/27dt3TiddGd)

If you already have an account, you can skip this step

## 2、[Create Cloud Server DC2](https://app.didiyun.com/#/dc2/add)

*We started to create a cloud host and installed it with one-click mirroring of Nightingale. If you are not familiar with cloud resource operations, you can [click to view demo video](https://s3-gz01.didistatic.com/n9e-pub/video/n9e-install-didiyun.mp4)*

If the payment method is "Period", it will be cheaper, and the configuration will not be repeated one by one. Note that in **Mirror **, select "One-click Deployment Mirror", and then select Nightingale's mirror, as shown in the figure:

![](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-didiyun-image-choose.png)

## 3、Access the platform and change the password

After the virtual machine is started, the built-in initialization script in the image will automatically install each module component of Nightingale. Wait about 2 minutes to complete the installation. Then enter the eip of the virtual machine in the browser to access the platform page. Log in with a root account, the password is also root, do not check the LDAP option. After successful login, immediately click on the personal settings in the upper right corner of the platform, modify the root password, and experience it immediately :-)
## 4、Some extra information

The mirror comes with mysql, nginx, redis, nginx listening at 80, if access is not available, you need to check the security group rules. Nightingale is deployed in the `/home /n9e` directory, the configuration file is in the` /home /n9e /etc` directory, you can see mysql.yml, you can find the root password of mysql.

