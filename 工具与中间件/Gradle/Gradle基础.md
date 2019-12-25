# Gradle基础

## 入门

### 基本介绍

+ Ant：2000 年 Apache 推出的纯 Java 编写构建工具，通过 xml[build.xml] 文件管理项目

  + 优点：使用灵活，速度快快于 gradle 和 maven

  + 缺点：Ant 没有强加任何编码约定的项目目录结构，开发人员需编写繁杂 XML 文件构建指令，对开发人员是一个挑战。

+ Maven：2004 年 Apache 组织推出的再次使用 xml 文件 pom.xml 管理项目的构建工具。

  + 优点：遵循一套约定大于配置的项目目录结构，使用统一的 GAV 坐标进行依赖管理，**侧重于包管理**。

  + 缺点：项目构建过程僵化。配置文件编写不够灵活、不方便自定义组件，构建速度慢于gradle。

+ Gradle：2012 年 Google 推出的基于 Groovy 语言的全新项目构建工具，集合了 Ant 和 Maven 各自的优势。

  + 优点：集Ant脚本的灵活性+Maen约定大于配置的项目目录优势支持多种远程仓库和插件，**侧重于大项目构建**。

  + 缺点：学习成本高、资料少、脚本灵活、版本兼容性差等。

Gradle 是项目构建工具，而 Maven 侧重于 Jar 包管理

![image-20230102083729812](img\image-20230102083729812.png)

### Gradle 安装

配置好环境变量

+ GRADLE_HOME：具体某一个gradle版本的路径，如`D:\software\grad-7.4.1`
+ GRADLE_USER_HOME：gradle 的本地仓库位置，如`E:\Config\.gradle`

### 项目结构

![image-20230101200421417](img\image-20230101200421417.png)

### 创建项目

先创建一个文件夹，然后cd进去，使用命令行`gradle init`

1、常用指令

![image-20230102090528501](img\image-20230102090528501-1672621536957-5.png)

2、修改下载源

在 GRADLE_USER_HOME 目录下新建一个 `init.gradle` 文件，内容如下：

```gradle
allprojects {
	repositories {
		mavenLocal()
		maven { name "Alibaba" ; url "https://maven.aliyun.com/repository/public" }
		maven { name "Bstek" ; url "https://nexus.bsdn.org/content/groups/public/" }
		mavenCentral()
	}
	buildscript {
		repositories {
			maven { name "Alibaba" ; url 'https://maven.aliyun.com/repository/public' }
			maven { name "Bstek" ; url 'https://nexus.bsdn.org/content/groups/public/' }
			maven { name "M2" ; url 'https://plugins.gradle.org/m2/' }
		}
	}
}
```

阿里云仓库地址：https://developer.aliyun.com/mvn/guide

## 进阶

### 生命周期

![image-20230103093449717](img\image-20230103093449717.png)

+ **Initialization 阶段**：主要目的是初始化构建，它又分为两个子过程，一个是执行 Init Script，另一个是执行 Setting Script。
+ **Configuration 阶段**：这个阶段开始加载项目中所有模块的 Build Script。所谓 "加载" 就是执行 build.gradle 中的语句，根据脚本代码创建对应的 task， 最终根据所有 task 生成由 Task 组成的有向无环图
+ **Execution 阶段**：这个阶段会根据上个阶段构建好的有向无环图，按着顺序执行 Task【Action 动作】。

### setting 文件

一个子工程只有在 setting 文件中配置了才会被 gradle 识别，这样在构建的时候才会被包含进去。

```gradle
//根工程项目名
rootProject.name = 'root'
//包含的子工程名称
include 'subject01'
include 'subject02'
include 'subject03'
//包含的子工程下的子工程名称
include 'subject01:subproject011'
include 'subject01:subproject012'
```

项目名称中 `:` 代表项目的分隔符，类似路径中的 `/`。如果以 `:` 开头则表示相对于 root project 。然后 Gradle 会为每个带有 build.gradle 脚本文件的工程构建一个与之对应的 Project 对象。

### Task

#### 任务执行

1、常见的任务

gradle build：构建项目:编译、测试、打包等操作

gradle run：运行一个服务,需要 application 插件支持，并且指定了主启动类才能运行

gradle clean：请求当前项目的 build 目录

gradle init：初始化 gradle 项目使用

gradle wrapper：生成 wrapper 文件夹的。

gradle wrapper 升级 wrapper 版本号：gradle wrapper --gradle-version=4.4

gradle wrapper --gradle-version 5.2.1 --distribution-type all：关联源码用

2、性能选项（可以在 gradle.properties 中配置）

```properties
# JVM 堆大小
org.gradle.jvmargs=-Xmx5120m -XX:MaxPermSize=1280m -Dfile.encoding=UTF-8
# 通过开启守护进程，下一次构建的时候，将会连接这个守护进程进行构建，而不是重新fork一个gradle构建进程
org.gradle.daemon=true
# 按需加载
org.gradle.configureondemand=true
# 并行编译
org.gradle.parallel=true
# 开启 gradle 缓存
org.gradle.caching=true
# cpu核数
max-workers=8
```

更多详情参考文档 https://docs.gradle.org/current/userguide/command_line_interface.html
