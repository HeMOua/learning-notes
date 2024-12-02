# Docker 打包镜像

>导出容器：docker export [容器id] > [name.tar]
>
>导出镜像：docker save [镜像id/标签] > [name.tar]
>
>加载：docker load < [name.tar]

## 一、基于docker容器Commit命令打包

第一步：拉项目镜像

```
docker pull  镜像名
```

第二步：基于镜像生成docker（端口映射，数据卷挂载，定制化自启动和root登录权限）

```shell
docker run -dit -p 222:22 -p 8081:8080 -p 80:80 -p 3307:3306 -p 6380:6379 -p 15673:15672 -p 22123:22122 -p 23001:23000 -v /d/usr:/home -v /c/Windows/identification:/data/identification --privileged --restart=always -h 用户名 --name=容器名 镜像名:版本号  /bin/bash  /etc/rc.d/enable
```

第三步：进入docker，添加或修改docker（这里看需求修改：我个人添加表sql，备份文件，脚本文件迁移到docker上）用到了以下指令：

1、进入docker

```shell
docker start 容器名
docker exec -it 容器名  bash
```

2、宿主机复制文件到docker（当然也可以用Xftp）：

```shell
docker cp 文件路径 容器长ID:docker容器中的路径
```

3、连接数据库执行sql文件

①连接MySQL：mysql -u用户 -p密码

②选择数据库：use 数据库名;

③执行sql文件：source 脚本文件全路径（/data/xxx.sql）

第四步：docker commit命令生成副本镜像（先stop容器）

```shell
docker stop 容器名
docker commit -m "备注" -a "修改人" 容器id  镜像名:新版本号
```

第五步：验证副本镜像（重复第二步骤、第三步骤，查看自己的修改）

第六步：docker push命令上传至Docker Hub上

```shell
docker push 镜像名:新版本号
```

第七步：登录Docker Hub查看自己的提交



## 二、基于docker快照打包镜像（export导出，import导入命令）

第一步：基于容器导出tar包（这里LZ的tar包名：luntek-ic-platform3d5.tar），export命令(-o：指向导出tar文件，也可以用”>“大于号替代)

```shell
# container表示容器id或容器名
docker export [options] container
```

示例如下： 

> 如果容器启动的，则需要stop容器，确保在创建快照时，容器内部的状态是稳定的
>
> docker stop  容器ID

```shell
# 使用 export 导出容器，使用的是容器 id
docker export > luntek-ic-platform3d5.tar 容器ID
# 或者
docker export -o luntek-ic-platform3d5.tar 容器ID 
```

第二步：将tar包解压导入为镜像（这里LZ的镜像名：luntek/ic-platform:3.5）

注意：推荐使用方式三，因为它更符合现代的Docker镜像管理实践。

```shell
docker import [options] file|URL|- [REPOSITORY[:TAG]]
```

示例如下：  

> 方式一：docker import方式在2022年的时候就已过时
>
> ```shell
> docker import luntek-ic-platform3d5.tar luntek/ic-platform:3.5
> ```

> 方式二：管道方式解压
>
> ```shell
> cat luntek-ic-platform3d5.tar | docker import - luntek/ic-platform:3.5
> ```

> 方式三：load加载压缩文件，然后docker tag 命令为镜像指定新的名字和版本号。
>
> ```shell
> docker load -i luntek-ic-platform3d5.tar
> 
> docker tag <IMAGE_ID> luntek/ic-platform:3.5
> ```

第三步：docker上传至docker hub

```shell
sudo docker push luntek/ic-platform:3.5
```

## 三、基于镜像导出导入（save导出，load导入命令） 

第一步：基于镜像(可多个镜像)中导出文件（这里LZ的文件名：luntek-ic-platform3d5.tar，镜像名：luntek/ic-platform:3.5），save命令(-o：指向导出文件，也可以用”>“大于号替代)

 注意：推荐使用方式二，因为它更符合现代的Docker镜像管理实践。

```shell
# images [images...] 可以有多个 images 镜像
docker save [options] images [images...]
```

示例如下：  

> 方式一：2023年8月Docker官方更新了save命令，如下已过时
>
> ```shell
> docker save -o /root/luntek-ic-platform3d5.tar luntek/ic-platform:3.5
> # 或者
> docker save > /root/luntek-ic-platform3d5.tar luntek/ic-platform:3.5
> ```

> 方式二：
>
> ```shell
> docker save  luntek/ic-platform:3.5 > /root/luntek-ic-platform3d5.tar
> # 或者
> docker save  镜像ID > /root/luntek-ic-platform3d5.tar
> ```

第二步：基于文件解压导入镜像（这里LZ的绝对路径文件名：/root/luntek-ic-platform3d5.tar），docker load命令（-i：指向解压导入文件，也可以用”<“小于号替代）

示例如下：  

```shell
docker load -i /root/luntek-ic-platform3d5.tar
# 或者
docker load < /root/luntek-ic-platform3d5.tar
```

第三步：docker上传至docker hub

```shell
sudo docker push luntek/ic-platform:3.5
```

