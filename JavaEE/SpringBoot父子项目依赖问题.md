# SpringBoot父子项目依赖问题

## parent方式

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging>
    <modules>
        <module>codegen-generator</module>
    </modules>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
        </dependencies>
        ...
    </dependencyManagement>
</project>
```

使用此种方式，写在 dependencyManagement 中的依赖会覆盖 springboot parent中预定好的版本，因此如果使用父子模块，那么springboot parent中存在的依赖就不要在父 pom 的 dependencyManagement 中定义了，直接写在子项目的 pom 中。

还有要**注意**的是，子模块需要定义在`dependencyManagement`中