# Redis排错指南

## 1、版本问题

配置如下

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>

<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.4.2.RELEASE</version>
</dependency>
```

Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'jedisConnectionFactory' defined in class path resource [spring_redis.xml]: Invocation of init method failed; nested exception is java.lang.NoSuchMethodError: **redis.clients.jedis.JedisShardInfo.setTimeout(I)V**

出现如上错误，基本是因为`redis.clients`的**版本太高了**

jedis的版本号换位2.6.2以下就好了

```xml
 <dependency>
     <groupId>redis.clients</groupId>
     <artifactId>jedis</artifactId>
     <version>2.6.1</version>
 </dependency>
```

这样配置就没错了

## 2、java.io.IOException: 远程主机强迫关闭了一个现有的连接。

可能用的是老的window版本，换成 linux 新版本就没问题了