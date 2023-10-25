# Java环境

1、下载jdk

[下载地址](https://www.oracle.com/java/technologies/downloads/#java8)，`jdk-8u321-linux-x64.tar.gz`

2、解压

```shell
sudo mkdir -p /usr/local/java/jdk1.8_321 # 创建目录
# sudo tar -zxvf jdk-8u321-linux-x64.tar.gz -C /usr/local/java/jdk1.8_321
# 或者，推荐下面的，貌似会创建父文件夹
sudo tar -zxvf jdk-8u321-linux-x64.tar.gz -C /usr/local/java
```

3、配置环境变量

输入`vim /etc/profile`进入编辑，在文件末尾添加

```shell
export JAVA_HOME=/usr/local/java/jdk1.8_321
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

4、使环境变量生效并测试JDK

```shell
source /etc/profile
java -version
```



如果 `java -version` 的结果错误，八成是因为路径写错了，看看是不是将 jdk1.8.0 写成了 jdk1.8
