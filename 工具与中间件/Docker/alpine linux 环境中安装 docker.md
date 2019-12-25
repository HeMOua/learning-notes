# **alpine linux 环境中安装 docker**

- alpine linux 是一个基于安全的轻量级 Linux 发行版，基于 musl libc 和 busybox。alpine linux 由于简单安全，非常适合 docker 的 linux 发行版，并且被 docker 官方所推荐用来取代 ubuntu。它不但非常小，仅有 5m 左右，而且经过优化可以在 RAM 中运行
- 既然这么省资源，那么本文就尝试在虚拟机中安装 alpine，然后安装 docker
- [官方文档](https://wiki.alpinelinux.org/wiki/Main_Page)

## 下载

- alpinelinux`https://www.alpinelinux.org/downloads/`，这里选择`VIRTUAL`，它与标准版相似、瘦下来的内核、针对虚拟系统进行了优化。
- [下载地址链接](http://dl-cdn.alpinelinux.org/alpine/v3.9/releases/x86_64/alpine-virt-3.9.4-x86_64.iso)
- 下载得到的文件是`alpine-virt-3.9.4-x86_64.iso`，大小 35m

## VMware 中安装

- 按道理 VirtualBox 也是一样的，VMware 的新建虚拟机的设置中，操作系统类型选择`Linux`，版本选择`其他Linux 4.x 或更高版本内核 64 位`
- 其他步骤按照提示来就行了，问题不大。新建完后启动虚拟机

## 设置 alpine

- 启动之后进入安装步骤
- 具体步骤参考 https://wiki.alpinelinux.org/wiki/Install_to_disk
- 可参考博客：[Alpine linux 硬盘安装](https://blog.csdn.net/csdn_duomaomao/article/details/76053229)、[alpine linux docker 安装体验](https://blog.csdn.net/ochinchina_cn/article/details/58593271)

## apk 介绍

- Alpine Linux 的软件包是经过数字签名的 tar.gz 存档，包含程序，配置文件和依赖关系元数据。它们具有扩展名.apk，通常称为“a-packs”。包存储在一个或多个存储库中。
- 相关文档，包含常用命令及介绍。https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management
- [包搜索地址](https://pkgs.alpinelinux.org/packages)
- 有些包可能在社区的源才有，地址：http://dl-cdn.alpinelinux.org/alpine/latest-stable/community

## 安装 docker

- Docker 包位于“社区”存储库中，因此如果 apk 添加失败且具有不可满足的约束，则需要编辑`/etc/apk/repositories`文件以添加（或取消注释）一行，如：http://dl-cdn.alpinelinux.org/alpine/latest-stable/community
- 更新索引存储库`apk update`
- 开始安装`apk add docker`
- 要在引导时启动 Docker 守护程序，请运行：`rc-update add docker boot`
- 然后手动启动 Docker 守护程序，运行：`service docker start`
- 参考[alpinelinux Docker](https://wiki.alpinelinux.org/wiki/Docker)

## 运行

- 启动`service docker start`
- `docker run hello-word`

```bash
localhost:/etc/docker# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:0e11c388b664df8a27a901dce21eb89f11d8292f7fca1b3e3c4321bf7897bffe
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## 问题

- 上面安装完 docker 之后，使用命令`docker -v`可查看版本，但是容器相关命令出错

```bash
localhost:~# docker ps
Cannot connect to the Docker daemon at unix:///var/rundocker.sock. Is   the docker daemon running?
```

- 通过命令`service docker start`也启动 docker，`/var/run/docker.sock 也确实存在。
- 折腾一番之后，查看日志`cat ./var/log/docker.log`，得到关键信息`Error starting daemon: Devices cgroup isn't mounted`。

```bash
cat ./var/log/docker.log
```

- 最后看了官方文档得以解决，https://wiki.alpinelinux.org/wiki/Docker
- 关键点还是`cgroup filesystem`的问题，`cgroup`是什么请自行查询

```bash
localhost:/etc/docker# rc-update add docker boot
 * service docker added to runlevel boot
localhost:/etc/docker# service docker start
 * Mounting cgroup filesystem ...
 * Starting docker ...
```

- 所以整个问题的根源是`cgroup filesystem`没有挂载，在执行`rc-update add docker boot`后在启动就好了

## 总结

- 多次总结，使用一款产品，第一件事就是找到文档，文档实在晦涩难懂的情况，那么可能有翻译版或者国内版，毕竟官方的才是最权威的
- 其次是每个人用起来遇到的环境可能不一样，遇到的问题也就不一样，但所有的解决思路中，查看日志这一条是我觉得始终应该第一考虑的，它很明白告诉你，出错了，出了什么错，可能还有出错的具体位置以及建议解决方案
- end