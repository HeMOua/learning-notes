# 整合Mybatis

首先引入依赖以及数据库驱动

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>

<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

在主类上添加Mapper Java类路径

```java
@MapperScan(basePackages = "com.example.mapper")
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

添加 Mapper xml 路径配置以及数据库连接配置

```yaml
mybatis:
  type-aliases-package: com.example.forum.entity
  mapper-locations: mapper/**.xml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/forum?characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
    username: root
    password: root
```

