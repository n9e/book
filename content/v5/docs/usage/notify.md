---
title: "告警通知"
linkTitle: "告警通知"
date: 2021-04-01
description: >
  告警消息发送，为了适配各种媒介，我们直接使用python脚本 etc/script/notify.py 来对接，本文将对告警信息发送的配置方式及notify.py原理进行介绍
---
在《[夜莺介绍](https://n9e.didiyun.com/docs/intro/)》一文中，我们介绍了夜莺的告警信息通知 notify.py 脚本来实现的，本文详细讲解下 notify.py 的实现原理，以及如何在页面上配置用户的联系方式，来支持多种通知渠道


### notify.py原理

因为通知渠道多种多样（短信、邮件、电话、微信、钉钉、slack、飞书、jira、公司自有im等等），夜莺没法内置支持所有的通知渠道，所以需要一个非常灵活易于扩展的方式，最灵活易于扩展的方式显然是代码。所以夜莺发送告警通知的时候，会使用notify.py脚本来发送，这个脚本放在etc/script目录下，大家可以自行修改。n9e-server进程生成告警事件之后，会把告警事件的内容整理成一个json字符串，通过stdin的方式传给notify.py，所有的发送逻辑都在notify.py里完成。各位运维研发的小伙伴，对于阅读并修改python脚本应该是比较得心应手的，所以大胆改造这个脚本吧，欢迎PR，让notify.py支持更多发送媒介。

notify.py收到告警信息之后，会将告警格式化为一个有缩进的json写入一个临时文件，为后续排查问题使用，临时文件的命名规则是 `${rule_id}_${event_id}_${trigger_time}`。这个临时文件会放到哪个目录呢？请阅读notify.py自己找答案。所以，只要看到临时文件的生成，就说明告警逻辑是没有问题的。如果有某个媒介没有通知成功，那就要仔细查看notify.py的代码了。如果觉得notify.py的代码也没问题，那就要怀疑通道媒介是否做了限制之类的，比如钉钉，就有每分钟发送条数的限制。

另外notify.py具体如何支持多种通知媒介呢？其实要想把消息发出去，只要知道发送对象的联系方式和发送通道就可以了，发送对象可以从json对象的event.users中获取，发送通道可以从 event.notify_channels 中获取，具体如何配置，下面会详细介绍。


### 邮件发送配置

目前notify.py 提供了发送邮件和钉钉群消息的能力，下面介绍下如何接入使用。发送邮件的话，首先需要修改notify.py中的邮件网关配置：

```python
mail_host = "smtp.163.com"
mail_port = 994
mail_user = "ulricqin"
mail_pass = "password"
mail_from = "ulricqin@163.com"
```

修改完成之后，在个人信息-邮箱的地方，填上自己的邮箱

![image.png](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-v5-user-profile.png)

如果邮件网关配置的没问题的话，当告警事件触发后，就可以收到告警邮件通知了。如果没有收到，可以依次检查页面上是否生成未恢复的告警事件，检查上文提到的临时文件是否生成，检查邮件网关配置，检查邮件发送代码逻辑，也可以自己写个邮件测试小脚本测试下邮件发送的方法逻辑。

### 钉钉群消息发送配置

首先创建钉钉机器人（钉钉有个安全设置，选择关键字匹配，关键字写20），拿到钉钉机器人的token，配置到某个user的联系方式中，然后把这个user作为告警接收人即可。

![image.png](https://s3-gz01.didistatic.com/n9e-pub/image/n9e-v5-user-contacts.png)


### 支持其他通知媒介

这里以发送微信群消息为例，主要分三个步骤。

第一，修改 server.yml ，contactKeys增加一项Wecom Robot Token，这样在用户的更多联系方式中就会自动展示这个字段，notifyChannels中增加wecom。默认提供的 server.yml 已经提供了企业微信的这个配置了，所以大家维持默认的不用动即可。


```yaml
contactKeys:
  - label: "Wecom Robot Token" # User的更多联系方式的下拉框中，微信机器人方式的label部分
    key: wecom_robot_token # User的更多联系方式的下拉框中，微信机器人方式的key部分，需要和notify.py中获取用户此联系方式的key相同
  - label: "Dingtalk Robot Token"
    key: dingtalk_robot_token
    
notifyChannels:
  - email
  - sms
  - voice
  - dingtalk 
  - wecom # 通知渠道，wecom代表要启用企业微信通知媒介
```
   
第二，个人联系方式中会看到Wecom Robot Token（因为contactKeys有此配置），把企业微信机器人的token填上

第三，修改notify.py，编写发送逻辑。notify.py中根据wecom这个notifyChannels找到对应的发送方法，然后从users的wecom_robot_token字段拿到用户的企业微信token，就可以调用微信的接口发送通知了。
