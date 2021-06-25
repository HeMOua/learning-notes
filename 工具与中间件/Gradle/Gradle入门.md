# Gradle入门

## 介绍

2012年基于Ant和Maven产生的Gradle，弥补了Ant和Maven的不足，带来了一些更高效的特点。它使用一种基于Groovy的特定领域语言（DSL]来声明项目设置，抛弃了基于XML的各种繁琐配置。面向Java应用为主。当前其支持的语言限于Java，Groovy和Scala，计划未来将支持更多的语言。

## 安装

http://services.gradle.org/distributions/

1、目录结构

+ bin
  + gradle.bat 构建项目的一个重要脚本
+ caches：缓存jar
+ daemon / init.d / lib：gradle自身所依赖的环境

2、配置环境变量

+ **GRADLE_HOME**：xxx
+ path：%GRADLE_HOME%/bin

## Groovy 简单语法

### 语句

```groovy
println("hello groovy");
```

+ 可以省略`;`
+ 可以省略括号

如下

```groovy
println "hello groovy"
```

### 定义变量

```groovy
def i = 10
```

+ def 定义弱类型变量，groovy 会自动根据情况来给变量赋予对应的类型

### list

```groovy
def list = ['a', 'b']
list << 'c'
println list.get(2) // 'c'
```

### map

```groovy
def map = ['key1': 'value1']
map.key2 = val2
println map.get('key2')
```

### 闭包

> 闭包就是一段代码块，在gradle中，主要是把闭包当参数来使用。

1、不带参数闭包

```groovy
// 1. 定义一个闭包（代码块）
def b1 = {
	println "hello b1"
}

// 2. 定义方法，参数为闭包类型
def method(Closure closure) {
	closure()
}

// 3. 调用方法
method(b1) // 输出 hello b1
```

2、带参数

```groovy
// 1. 定义一个闭包（代码块）
def b1 = {
	v -> println v
}

// 2. 定义方法，参数为闭包类型
def method(Closure closure) {
	closure("xiao mi")
}

// 3. 调用方法
method(b1) // 输出 hello b1
```

## build.gradle 配置文件

```groovy
// 类似 maven 的坐标
group 'com.hemou'
version '1.0'

// 使用java的编译方式
apply plugin: 'java'

// 指定所使用的仓库，mavenCentral() 表示使用中央仓库，此刻项目中所需要的jar包都会默认从中央仓库中下载到本地指定目录
repositories {
    // 表示先从maven本地仓库查找，找不到的再从中央仓库下载
    mavenLocal()
    mavenCentral()
}

dependencies {
    implementation fileTree(dir: "libs", include: ["*.jar"])
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'

}

```

+ 环境变量
  + **GRADLE_USER_HOME**：设置为maven本地仓库的位置

## gradle 命令

在IDEA右侧gradle的工具栏中，

+ Tasks
  + build
    + jar：可以用将项目打包成jar，生成的jar在 build -> libs 下

## web 项目

```groovy
group 'com.hemou'
version '1.0'

apply plugin: 'java'
apply pulgin: 'war' // 只要多添加一行这个，即可
```

## 多模块项目

首先父项目的`build.gradle`中的所有配置需要移动到  `allprojects` 中，如下

```groovy
allprojects {
    group 'com.hemou'
    version '1.0'

    apply plugin: 'java'
    ...
}
```

如此一来，所有的子模块都会继承父模块的所有配置，仅需在子模块中添加有所区别的配置即可



