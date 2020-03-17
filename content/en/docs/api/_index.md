---
title: "API"
linkTitle: "API"
weight: 9
date: 2020-03-17
description: >
  本节会介绍Nightingale对外提供的API，您可以利用这些API做二次开发，与内部系统相集成。页面上面的所有操作，都有API提供
---

## 前言

1、所有接口的返回均为以下格式，错误信息会写入到`err`字段中，status code全部是200，除非服务端故障了

```json
{
    "dat": {},
    "err": "error msg"
}
```

2、`/api/portal`相关接口会进行鉴权，采用basic auth方式

```bash
# 比如使用curl可以这么测试：
curl -u root:1234 localhost/api/portal/self/profile
```

上例中假设采用root账号（密码1234）来测试。实际生产环境，我们可以创建几个虚拟账号，权限设置为超管，用这几个虚拟账号来作为调用方鉴权账号

3、Nightingale各个模块前面会有一个nginx，所以所有接口的地址默认为nginx的地址。如果想直接调用后端模块，也未尝不可，注意做好负载均衡和失败重试即可。

4、monapi相关的接口，可以参考文件：`src/modules/monapi/http/routes/routes.go`，这个文件定义了所有接口的path和对应的实现方法，如果觉得接口不够想补充，可以提交PR :-) 而查询索引数据的接口是index模块提供，api路径以`/api/index`开头，查询历史监控数据的接口是transfer模块提供，api路径以`/api/transfer`开头

下面我们会把一些重要接口做简要介绍，其他接口请直接查阅源码，毕竟，代码是不会骗人的