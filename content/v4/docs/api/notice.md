---
title: "自定义告警通知接口"
linkTitle: "告警通知"
weight: 20
date: 2020-03-17
description: >
  如果贵司有自己的短信、电话通知接口，可以接入夜莺，只要在rdb里做简单配置即可
---


系统默认支持邮件、企业微信、钉钉接入，如果要使用公司内部发送通知的通道，可以采用两种扩展机制：

- 1、实现api接口，配置到rdb里，发送告警时rdb会自动调用api接口
- 2、编写发送脚本，放到指定位置，按照规定的命名方式命名，rdb会在发送告警时自动调用相应脚本


对于api的对接方式，夜莺会给自定义接口发送如下body数据（POST方式）：

```json
{
	"tos": ["xx1","xx2"],
	"subject": "test",
	"content": "alert test xxxx"
}
```


# 一、IM

## 1. API

- **方法**： POST
- **header**: application/json
- **body**: body数据
- **配置方式**：rdb.yml -> sender -> im(way:api;api:http://xxxx:8890/im)

## 2. Shell

- **参数1**：字符串  告警接受人，多个逗号隔开
- **参数2**：字符串  告警内容
- **配置方式**：rdb.yml -> sender -> im(way:shell)
- **Shell脚本名称与路径**：n9e-rdb二进制同级下的script目录下send_im


## 二、MAIL

### 1. API

- **方法**： POST
- **header**: application/json
- **body**: body数据
- **配置方式**：rdb.yml -> sender -> mail(way:api;api:http://xxxx:8891/mail)

### 2. Shell

- **参数1**：字符串  告警接受人，多个逗号隔开
- **参数2**：字符串  告警标题
- **参数3**：字符串  写入告警内容的文件路径
- **配置方式**：rdb.yml -> sender -> mail(way:shell)
- **Shell脚本名称与路径**：n9e-rdb二进制同级下的script目录下send_mail


## 三、短信

### 1. API

- **方法**： post
- **header**: application/json
- **body**: body数据
- **配置方式**：rdb.yml -> sender -> sms(way:api;api:http://xxxx:8892/sms)

### 2. Shell

- **参数1**：字符串  告警接受人，多个逗号隔开
- **参数2**：字符串  告警内容
- **配置方式**：rdb.yml -> sender -> sms(way:shell)
- **Shell脚本名称与路径**：n9e-rdb二进制同级下的script目录下send_sms

## 四、电话语音

### 1. API

- **方法**： post
- **header**: application/json
- **body**: body数据
- **配置方式**：rdb.yml -> sender -> voice(way:api;api:http://xxxx:8893/voice)

### 2. Shell

- **参数1**：字符串  告警接受人，多个逗号隔开
- **参数2**：字符串  告警内容
- **配置方式**：rdb.yml -> sender -> voice(way:shell)
- **Shell脚本名称与路径**：n9e-rdb二进制同级下的script目录下send_voice


