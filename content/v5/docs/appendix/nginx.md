---
title: "配置nginx"
linkTitle: "配置nginx"
date: 2021-06-19
description: >
  在夜莺v5里，前端代码可以直接使用n9e-server来serve，不需要nginx，不过有的场景是不希望把n9e-server的端口暴露出来的，比如 《[API调用](/docs/appendix/api/)》章节提到的接口安全问题，此时就需要nginx了
---


下面给出nginx的典型配置，大家也可以加一下gzip压缩之类的提升性能，这个就由各位自由发挥了。


```
server {
    listen       80;
    server_name  _;

    root   /opt/n9e/pub;

    location / {
        root /opt/n9e/pub;
        try_files $uri /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
    }
}
```