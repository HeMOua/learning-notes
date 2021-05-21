# Maven

## 多模块

1、首先创建一个maven工程，作为父项目，注意`packaging`标签

> `packaging`标签指定打包的方式，默认为jar，如果pom文件中没有`packing`标签那么默认就是打jar

> maven父工程必须遵顼以下两点要求
>
> + `packaging`标签的文本内容必须设置为pom
> + 把src删除掉，（maven项目三元素src、target和pom.xml）

> pom 是项目对象模型（Project Object Module），该文件可以被子模块继承
>
> maven多模块管理，其实就是让它的子模块的pom文件来继承父模块的pom工程

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>cn.hemou</groupId>
  <artifactId>ssm</artifactId>
  <version>0.0.1-SNAPSHOT</version>
    
  <packaging>pom</packaging>
 
</project>
```

2、然后再创建一个项目，指定父工程的 groupId、version、artifactId和relativePath，此时子工程的groupId和version就可以去掉了

3、版本管理

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>cn.hemou</groupId>
  <artifactId>ssm</artifactId>
  <version>0.0.1-SNAPSHOT</version>
    
  <packaging>pom</packaging>
 
  <dependencies></dependencies>
    
  <dependencyManagement>
    <dependencies></dependencies>
  </dependencyManagement>
</project>
```

所有写在父工程`dependencies`里的依赖都会被子工程继承拥有

但是这样可能会产生冗余，有时子工程可能不需要这个依赖，那么就可以用`dependencyManagement`来进行版本管理，当子模块需要这个依赖时可以在自己的pom.xml中添加配置，只不过不同的是版本号可以不写。

当子模块版本号不写时，则用父模块指点的模块，否者按照子模块指点的版本引入