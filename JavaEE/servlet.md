# Servlet

可以使用`@WebServlet`等注解的jar依赖

```xml
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-servlet-api</artifactId>
    <version>8.5.65</version>
    <scope>provided</scope>
</dependency>
```



JSTL 相关依赖

```xml
<dependency>
    <groupId>javax.servlet.jsp.jstl</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
<dependency>
    <groupId>taglibs</groupId>
    <artifactId>standard</artifactId>
    <version>1.1.2</version>
</dependency>
```



注意`isELIgnored`

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
  ``pageEncoding="UTF-8" isELIgnored="false"%>
```
