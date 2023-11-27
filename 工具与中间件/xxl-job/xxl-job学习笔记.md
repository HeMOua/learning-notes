# 分布式调度XXL-JOB

## 1. 概述

### 1.1 什么是任务调度

我们可以思考一下下面业务场景的解决方案：

+ 某电商平台需要每天上午10点，下午3点，晚上8点发放一批优惠券
+ 某银行系统需要在信用卡到期还款日的前三天进行短信提醒
+ 某财务系统需要在每天凌晨0:10分结算前一天的财务数据，统计汇总

以上场景就是任务调度所需要解决的问题

**任务调度是为了自动完成特定任务，在约定的特定时刻去执行任务的过程**

### 1.2 为什么需要分布式调度

使用Spring中提供的注解 `@Scheduled`,也能实现调度的功能

在业务类中方法中贴上这个注解，然后在启动类上贴上 `@Enablescheduling`注解

```java
@Scheduled(cron "0/20 * * * * ?"）
public void dowork（）{
	//dosomething
}
```

感觉 Spring 给我们提供的这个注解可以完成任务调度的功能，好像已经完美解决问题了，为什么还需要分布式呢？

主要有如下这几点原因：

1. 高可用：重机版的定式任务调度只能在一台机器上运行，如果程序或者系统出现异常就会导致功能不可用。
2. 防止重复执行：在单机模式下，定时任务是没什么问题的。但当我们部署了多台服务，同时又每台服务又有定时任务时，若不进行合理的控制在同一时间，只有一个定时任务启动执行，这时，定时执行的结果就可能存在混乱和错误了
3. 单机处理极限：原本1分钟内需要处理1万个订单，但是现在需要1分钟内处理10万个订单；原来一个统计需要1小时，现在业务方需要10分钟就统计出来。你也许会说，你也可以多线程、单机多进程处理。的确，多线程并行处理可以提高单位时间的处理效率，但是单机能力毕竟有限(主要是CPU、内存和磁盘)，始终会有单机处理不过来的情况。

### 1.3 XXL-JOB介绍

XXL-Job:是大众点评的分布式任务调度平台，是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展

大众点评目前已接入XL-JOB,该系统在内部已调度约100万次，表现优异。

目前已有多家公司接入 xxl-job，包括比较知名的大众点评，京东，优信二手车，360金融(360)，联想集团（联想），易信（网易）等等
官网地址 https://www.xuxueli.com/xxl-job

![img_Qohm](img/img_Qohm-1700529186932-3.png)

将调度行为抽象形成“调度中心”公共平台，而平台自身并不承担业务逻辑，“调度中心”负责发起调度请求。

将任务抽象成分散的 `JobHandler`，交由“执行器”统一管理，“执行器”负责接收调度请求并执行对应的 `JobHandler` 中业务逻辑。

因此，“调度”和“任务”两部分可以相互解耦，提高系统整体稳定性和扩展性；

## 2. 快速入门

### 2.1 初始化调度数据库

请下载项目源码并解压，获取 “调度数据库初始化SQL脚本” 并执行即可。

“调度数据库初始化SQL脚本” 位置为:

```
/xxl-job/doc/db/tables_xxl_job.sql
```

调度中心支持集群部署，集群情况下各节点务必连接同一个mysql实例;

如果mysql做主从,调度中心集群节点务必强制走主库;

### 2.2 编译源码

解压源码,按照maven格式将源码导入IDE, 使用maven进行编译即可，源码结构如下：

```
xxl-job-admin：调度中心
xxl-job-core：公共依赖
xxl-job-executor-samples：执行器Sample示例（选择合适的版本执行器，可直接使用，也可以参考其并将现有项目改造成执行器）
    ：xxl-job-executor-sample-springboot：Springboot版本，通过Springboot管理执行器，推荐这种方式；
    ：xxl-job-executor-sample-frameless：无框架版本；
```

### 2.3 配置部署调度中心

```
调度中心项目：xxl-job-admin作用：统一管理任务调度平台上调度任务，负责触发调度执行，并且提供任务管理平台。
```

#### 2.3.1 调度中心配置

调度中心配置文件地址：

```
/xxl-job/xxl-job-admin/src/main/resources/application.properties
```

调度中心配置内容说明：

```properties
### 调度中心JDBC链接：链接地址请保持和 2.1章节 所创建的调度数据库的地址一致
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=root_pwd
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
### 报警邮箱
spring.mail.host=smtp.qq.com
spring.mail.port=25
spring.mail.username=xxx@qq.com
spring.mail.password=xxx
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
spring.mail.properties.mail.smtp.socketFactory.class=javax.net.ssl.SSLSocketFactory
### 调度中心通讯TOKEN [选填]：非空时启用；
xxl.job.accessToken=
### 调度中心国际化配置 [必填]： 默认为 "zh_CN"/中文简体, 可选范围为 "zh_CN"/中文简体, "zh_TC"/中文繁体 and "en"/英文；
xxl.job.i18n=zh_CN
## 调度线程池最大线程配置【必填】
xxl.job.triggerpool.fast.max=200
xxl.job.triggerpool.slow.max=100
### 调度中心日志表数据保存天数 [必填]：过期日志自动清理；限制大于等于7时生效，否则, 如-1，关闭自动清理功能；
xxl.job.logretentiondays=30
```

#### 2.3.2 部署项目

如果已经正确进行上述配置，可将项目编译打包部署。

调度中心访问地址：http://localhost:8080/xxl-job-admin (该地址执行器将会使用到，作为回调地址)

默认登录账号 “admin/123456”, 登录后运行界面如下图所示。

![输入图片说明](img/img_6yC0.png)

至此“调度中心”项目已经部署成功。

#### 2.3.3 调度中心集群（可选）

调度中心支持集群部署，提升调度系统容灾和可用性。

调度中心集群部署时，几点要求和建议：

- DB配置保持一致；
- 集群机器时钟保持一致（单机集群忽视）；
- 建议：推荐通过nginx为调度中心集群做负载均衡，分配域名。调度中心访问、执行器回调配置、调用API服务等操作均通过该域名进行。

#### 2.3.4 Docker 镜像方式搭建调度中心

- 下载镜像

```shell
// Docker地址：https://hub.docker.com/r/xuxueli/xxl-job-admin/  (建议指定版本号)
docker pull xuxueli/xxl-job-admin
```

- 创建容器并运行

```shell
docker run -p 8080:8080 -v /tmp:/data/applogs --name xxl-job-admin  -d xuxueli/xxl-job-admin:{指定版本}
```

如需自定义 mysql 等配置，可通过 "-e PARAMS" 指定，参数格式 PARAMS="--key=value  --key2=value2" 

配置项参考文件：/xxl-job/xxl-job-admin/src/main/resources/application.properties

如需自定义 JVM内存参数 等配置，可通过 "-e JAVA_OPTS" 指定，参数格式 JAVA_OPTS="-Xmx512m" 

```shell
docker run -e PARAMS="--spring.datasource.url=jdbc:mysql://192.168.56.101:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai --spring.datasource.password=root" -p 8080:8080 -v /tmp:/data/applogs --name xxl-job-admin  -d xuxueli/xxl-job-admin:{指定版本}
```

### 2.4 配置部署执行器项目

### 2.5 运行 HelloWorld 程序

### 2.6 GLUE模式（Java）

### 2.7 执行器集群

#### 2.7.1 执行器集群搭建

创建两个相同 `xxl.job.executor.appname` 的执行器，（它们的ip或端口可能会有所不同）

#### 2.7.2  调度路由算法讲解

当执行器集群部署时，提供丰富的路由策略，包括：

1. FIRST（第一个）：固定选择第一个机器
2. LAST（最后一个）：固定选择最后一个机器；
3. ROUND（轮询）：依次的选择在线的机器发起调度
4. RANDOM（随机）：随机选择在线的机器；
5. CONSISTENT_HASH(一致性HASH)：每个任务按照Hash算法固定选择某一台机器，且所有任务均匀散列在不同机器上。
6. LEAST_FREQUENTLY_USED（最不经常使用）：使用频率最低的机器优先被选举；
7. LEAST_RECENTLY_USED（最近最久未使用）：最久未使用的机器优先被选举；
8. FAILOVER（故障转移）：按照顺序依次进行心跳检测，第一个心跳检测成功的机器选定为目标执行器并发起调度；
9. BUSYOVER（忙碌转移）：按照顺序依次进行空闲检测，第一个空闲检测成功的机器选定为目标执行器并发起调度；
10. SHARDING_BROADCAST（分片广播）：广播触发对应集群中所有机器执行一次任务，同时系统自动传递分片参数；可根据分片参数开发分片任务；

## 3. 分片功能讲解

### 3.1 案例需求讲解

假设有一个业务需要在指定时间给所有用户发送一条祝福短信，业务模拟如下

```java
@XxlJob("sendMsgHandler")
public void sendMsgHandler() throws InterruptedException {
    int userCount = 2000;
    System.out.println("任务开始时间：" + new Date() + "，任务数量：" + userCount);
    long startTime = System.currentTimeMillis();
    // 模拟2000个用户
    for (int i = 0; i < userCount; i++) {
        // 模拟发送消息
        TimeUnit.MILLISECONDS.sleep(10);
    }
    System.out.println("任务结束时间：" + new Date() + "，耗时：" + (System.currentTimeMillis() - startTime) + "ms");
}
```

比如我们的案例中有 2000+ 条数据，如果不采取分片形式的话，任务只会在一台机器上执行，这样的话需要 20+ 秒才能执行完任务

如果采取分片广播的形式的话，一次任务调度将会广播触发对应集群中所有执行器执行一次任务，同时系统自动传递分片参数；可根据分片参数开发分片任务；

获取分片参数方式：

```java
//可参考sample示例执行器中的示利任务"ShardingJobHandler"了解试用
int shardIndex Xx1JobHelper.getshardIndex();
int shardTotal Xx1JobHelper.getshardTotal();
```

通过这两个参数，我们可以通过求模取余的方式，分别查询，分别执行，这样的话就可以提高处理的速度。

之前 2000+ 条数据只在一台机器上执行需要 20+ 秒才能完成任务，分片后，有两台机器可以共同完成2000+ 条数，这样只要 10+ 就能完成任务

### 3.2 分片解决方案

```java
@Xx1Job ("sendMsgHandler")
public void sendMsgHandler() throws Exception {
    System.out.println("任务开始时间：" + new Date());
    int shardTotal = XxlJobHelper.getShardTotal();
    int shardIndex = XxlJobHelper.getShardIndex();
    List<UserMobilePlan> userMobilePlans = null;
    if(shardTotal == 1){
        // 如果没有分片就直接查询所有数据
        userMobilePlans = userMobilePlanMapper.selectAll();
    }else{
        userMobilePlans = userMobilePlanMapper.selectByMod(shardIndex,shardTotal);
        System.out.println("处理任务数量：" + userMobilePlans.size());
    }
    Long startTime System.currentTimeMillis(,
    userMobilePlans.forEach(item-> {
        try {
            // 模拟发送短信动作
            TimeUnit.MILLISECONDS.sleep (10),
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    System.out.println("任务结束时间：" + new Date());
    System.out.printIn("任务耗时："+ (System,currentTimeMillis() - startTime)+"毫秒");
}
```

