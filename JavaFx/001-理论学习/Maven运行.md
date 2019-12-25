## 依赖

```xml
<!-- javafx 库 -->
<dependency>
    <groupId>org.openjfx</groupId>
    <artifactId>javafx-fxml</artifactId>
    <version>11.0.2</version>
</dependency>
<dependency>
    <groupId>org.openjfx</groupId>
    <artifactId>javafx-controls</artifactId>
    <version>11.0.2</version>
</dependency>
<dependency>
    <groupId>org.openjfx</groupId>
    <artifactId>javafx-graphics</artifactId>
    <version>11.0.2</version>
</dependency>
```



## 插件

如果是 java1.8

```xml
<plugin>
    <groupId>com.zenjava</groupId>
    <artifactId>javafx-maven-plugin</artifactId>
    <version>8.8.3</version>
    <configuration>
        <mainClass>com.java.salary.App</mainClass>
        <appName>CodeGenerate</appName>
        <vendor>com.hemou</vendor>
    </configuration>
</plugin>
```

然后运行 `mvn clean jfx:run`



如果是 java11

```
<plugin>
    <groupId>org.openjfx</groupId>
    <artifactId>javafx-maven-plugin</artifactId>
    <version>0.0.8</version>
    <configuration>
        <mainClass>hellofx/org.openjfx.App</mainClass>
    </configuration>
</plugin>
```

然后运行 `mvn clean javafx:run`

如果在 eclipse 中需要将命令开始的 mvn 去掉