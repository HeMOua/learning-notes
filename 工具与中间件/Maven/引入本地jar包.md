# 引入本地jar包

## 简单使用

1、首先在项目根目录新建lib文件夹，将jar包放进去

2、pom.xml映入依赖

```xml
<dependency>
    <groupId>xxx</groupId>
    <artifactId>xxx</artifactId>
    <version>xxx</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/xxx.jar</systemPath>
</dependency>
```

- groupId：自定义
- artifactId：自定义
- version：自定义
- scope：必须是system
- systemPath：jar包的路径（idea编写的时候会有提示的）

## 处理打包

通过上述方式，在开发环境没有什么问题，不会存在什么包找不到等情况。但是，maven project部署一般打包发布，所以打包是需要额外配置的。

```xml
 <build>
   <resources>
    <resource>
      <directory>lib</directory>
      <targetPath>/BOOT-INF/lib/</targetPath>
      <includes>
        <include>**/*.jar</include>
      </includes>
    </resource>
   </resources>
 </build>
```

- directory：指定lib文件夹的位置，由于是相对工程根目录，所以直接写上lib即可
- targetPath：打包到的文件夹位置，写上`BOOT-INF/lib`即可，或者是`WEB-INF/lib`
  - 开头的斜杠`/`加不加都行，如果是mac的话写`./`
- includes：一般都是以jar结尾，就写`**/*.jar`



