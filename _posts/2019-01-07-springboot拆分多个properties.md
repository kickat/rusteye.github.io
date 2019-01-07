---
layout: post
title:  "springboot拆分多个properties"
date:   2019-01-07 00:00:00
categories: springboot
permalink: /archivers/springboot/properties
---

关于如何在springboot中使用多个配置文件初始化

<!--more-->

### 关于springboot使用不同properties

1、首先有这么几个文件

![1546848259513](/img/2019-01-07-properties/3.png)

application.properties

```
server.port=8090
server.tomcat.uri-encoding=utf-8
spring.application.name=demo
// 如果不存在app.env这个环境变量，则spring.profiles.active的值为prod
spring.profiles.active=${app.env:prod}
```

application-dev.properties
```
server.port=8090
server.tomcat.uri-encoding=utf-8
spring.application.name=demo

spring.profiles.active=${app.env:prod}
```
application-prod.properties

```
server.port=8092
server.tomcat.uri-encoding=utf-8
spring.application.name=demo

profile=prod_envrimont
spring.profiles.include=prodredis
```

application-prodredis.properties

```
ppp=111
```

2、测试取到的ppp的值

![1546849018851](/img/2019-01-07-properties/2.png)
结果：
![1546849119285](/img/2019-01-07-properties/3.png)


