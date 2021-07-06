---
title: "告警通知"
linkTitle: "告警通知"
date: 2021-05-25
description: >
  告警消息发送，为了适配各种媒介，我们直接使用python脚本 etc/script/notify.py 来对接，本文将对告警信息发送的配置方式及notify.py原理进行介绍
---
在《[夜莺介绍](https://n9e.didiyun.com/docs/intro/)》一文中，我们介绍了夜莺的告警信息通知 notify.py 脚本来实现的，本文详细讲解下 notify.py 的实现原理，以及如何在页面上配置用户的联系方式，来支持多种通知渠道


### notify.py原理
当夜莺生成告警事件之后，会调用notify.py脚本将告警信息发送出去，告警内容是怎么样传给notify.py的呢？是通过stdin的方式，代码详见 [alert/consume.go](https://github.com/didi/nightingale/blob/1f16bc9a7b788fed7d5a93f331f7ec0a99fba17b/alert/consume.go#L183) ，notify.py收到告警信息之后，会将告警格式化为一个有缩进的json写入一个临时文件，为了后续排查问题使用，临时文件的命名规则是 rule_id_event_id_trigger_time。那 notify.py 如何支持多种通知媒介呢？其实要想把消息发出去，只有知道发送对象的联系方式和发送通道就可以了，发送对象可以从event.users.contacts中获取，发送通道可以从 event.notify_channels 中获取，具体如何配置，下面会详细介绍


### 邮件发送配置
目前notify.py 提供了发送邮件和钉钉群消息的能力，下面介绍下如何接入使用
发送邮件的话，首先需要修改notify.py中的邮件[网关配置](https://github.com/didi/nightingale/blob/1f16bc9a7b788fed7d5a93f331f7ec0a99fba17b/etc/script/notify.py#L34)​
```python
mail_host = "smtp.163.com"
mail_port = 994
mail_user = "ulricqin"
mail_pass = "password"
mail_from = "ulricqin@163.com"
```
修改完成之后，在个人信息-邮箱的地方，填上自己的邮箱
![image.png](https://cdn.nlark.com/yuque/0/2021/png/626713/1625534454611-94884ce2-a7fa-4600-a1f8-0a6068d73f87.png)
如果邮件网关配置的没问题的话，当告警事件触发后，就可以收到告警邮件通知了

### 钉钉群消息发送配置
夜莺提供了支持多种消息媒介的能力，除了内置的邮箱和手机号之外，在 server.yml 的[配置文件]()中，可以配置额外的通知媒介，配置如下
```yaml
contactKeys:
  - label: "Wecom Robot Token" #页面上展示的通知媒介名称
    key: wecom_robot_token #存储到数据库中的通知媒介的key，需要和notify.py中获取用户此联系方式的key相同
  - label: "Dingtalk Robot Token"
    key: dingtalk_robot_token
    
 notifyChannels:
  - email
  - sms
  - voice
  - dingtalk #通知渠道，notify.py send方法中会用到
  - wecom
```
目前默认配置了微信机器人和钉钉机器人，如果想要增加其他方式，修改配置文件，然后重启 n9e-server 即可。
重启服务端之后，在创建用户的时候，就可以给用户添加在server.yml中配置的通知媒介了，如果是发送钉钉群消息的话，可以选择Dingtalk Robot Token
![image.png](https://cdn.nlark.com/yuque/0/2021/png/626713/1625535112342-9a781e15-5a05-47b3-ad32-984eee5bc37f.png)

当把demo03或者demo03所在的团队添加到某个告警策略的接收者之后，当告警触发的时候，对应的钉钉机器人所在的钉钉群就会收到告警信息，如下    
![image.png](https://cdn.nlark.com/yuque/0/2021/png/626713/1625535878366-028cb86a-e1bc-4e90-b49b-d496d3b3359e.png)

### 支持其他通知媒介
这里以发送微信群消息为例，主要分三个步骤    
第一，在 server.yml 中配置新的通知媒介联系字段和通知通道，然后重启服务器
​

```yaml
contactKeys:
  - label: "Wecom Robot Token" #微信机器人媒介名称
    key: wecom_robot_token #存储到数据库中的微信机器人的key，需要和notify.py中获取用户此联系方式的key相同
  - label: "Dingtalk Robot Token"
    key: dingtalk_robot_token
    
notifyChannels:
  - email #通知渠道，notify.py send方法中会用到
  - sms
  - voice
  - dingtalk 
  - wecom
```
   
第二，在创建用户的时候，以添加微信群消息举例，在更多联系方式的地方，把刚才新添加的 Wecom Robot Token 加上，点击确认，之后用户的数据结构contacts中会多出一个 wecom_robot_token:xxx，这个kv对即用户对应通知媒介的联系方式，notify.py脚本中，有多个名称为 send_xxx 的方法的，xxx是server.yml配置文件中 notifyChannels 配置的字段，以微信群消息举例，则是 wecom

![image.png](https://cdn.nlark.com/yuque/0/2021/png/626713/1625536155122-3a312896-e5b8-4a08-b469-8a0d973a300f.png)

```python
    @classmethod
    def send_wecom(cls, payload):
        print("send_wecom") #补充发送逻辑
```
   
第三， 到这一步，新通知媒介的方法名称知道了（notifyChannels.xxx），用户的联系方式也知道了(user.contacts[contactKeys.key])，就可以补充notify.py中send_xxx的方法了，脚本完成补充之后，在个人消息页面，配置对应新媒介的联系方式，就可以收到对应的消息了
![image.png](https://cdn.nlark.com/yuque/0/2021/png/626713/1625535411867-ef491b2b-bc0f-48f6-b6b6-17cb573f3f0f.png)
