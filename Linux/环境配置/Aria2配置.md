# Aria2配置

## 安装

```
sudo apt-get install aria2
```

## 配置

1、创建必要文件

```
mkdir ~/.aria2    #新建文件夹 
touch ~/.aria2/aria2.session    #新建session文件
chmod 777 ~/.aria2/aria2.session    #设置aria2.session可写 
vim ~/.aria2/aria2.conf    #创建配置文件
```

2、aria2.conf 配置文件

```properties

## 全局设置 ## ============================================================
# 日志
#log-level=warn
#log=/PATH/.aria2/aria2.log

# 后台运行
#daemon=true

# 下载位置, 默认: 当前启动位置(***)
dir=/home/***/Downloads/

# 从会话文件中读取下载任务(***) 都用绝对路径
input-file=/home/***/.aria2/aria2.session

# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件(***)
save-session=/home/***/.aria2/aria2.session

# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
save-session-interval=30

# 断点续传
continue=true

# 启用磁盘缓存, 0为禁用缓存, 需1.16以上版本, 默认:16M
#disk-cache=32M

# 文件预分配方式, 能有效降低磁盘碎片, 默认:prealloc
# 预分配所需时间: none < falloc ? trunc < prealloc
# falloc和trunc则需要文件系统和内核支持
# NTFS建议使用falloc, EXT3/4建议trunc, MAC 下需要注释此项
file-allocation=none
# 客户端伪装
user-agent=netdisk;5.2.6;PC;PC-Windows;6.2.9200;WindowsBaiduYunGuanJia
referer=http://pan.baidu.com/disk/home

# 禁用IPv6, 默认:false
disable-ipv6=true

# 其他
always-resume=true
check-integrity=true

## 下载位置 ## ============================================================
# 最大同时下载任务数, 运行时可修改, 默认:5
max-concurrent-downloads=10

# 同一服务器连接数, 添加时可指定, 默认:1
max-connection-per-server=10

# 最小文件分片大小, 添加时可指定, 取值范围1M -1024M, 默认:20M
# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则使用一个来源下载
min-split-size=10M

# 单个任务最大线程数, 添加时可指定, 默认:5
split=5

# 整体下载速度限制, 运行时可修改, 默认:0
#max-overall-download-limit=0

# 单个任务下载速度限制, 默认:0
#max-download-limit=0

# 整体上传速度限制, 运行时可修改, 默认:0
#max-overall-upload-limit=0

# 单个任务上传速度限制, 默认:0
#max-upload-limit=0

## RPC设置 ## ============================================================
# 启用RPC, 默认:false
enable-rpc=true

# 允许所有来源, 默认:false
rpc-allow-origin-all=true

# 允许非外部访问, 默认:false
rpc-listen-all=true

# 事件轮询方式, 取值:[epoll, kqueue, port, poll, select], 不同系统默认值不同
#event-poll=select

# RPC监听端口, 端口被占用时可以修改, 默认:6800
rpc-listen-port=6800

# 设置的RPC授权令牌, v1.18.4新增功能, 取代 --rpc-user 和 --rpc-passwd 选项
#rpc-secret=<TOKEN>

# 是否启用 RPC 服务的 SSL/TLS 加密,
# 启用加密后 RPC 服务需要使用 https 或者 wss 协议连接
#rpc-secure=true

# 在 RPC 服务中启用 SSL/TLS 加密时的证书文件,
# 使用 PEM 格式时，您必须通过 --rpc-private-key 指定私钥
#rpc-certificate=/path/to/certificate.pem

# 在 RPC 服务中启用 SSL/TLS 加密时的私钥文件
#rpc-private-key=/path/to/certificate.key

## BT/PT下载相关 ## ============================================================
# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
#follow-torrent=true

# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6999
listen-port=51413

# 单个种子最大连接数, 默认:55
#bt-max-peers=55

# 打开DHT功能, PT需要禁用, 默认:true
enable-dht=false

# 打开IPv6 DHT功能, PT需要禁用
#enable-dht6=false

# DHT网络监听端口, 默认:6881-6999
#dht-listen-port=6881-6999

dht-file-path=/opt/var/aria2/dht.dat
dht-file-path6=/opt/var/aria2/dht6.dat

# 本地节点查找, PT需要禁用, 默认:false
#bt-enable-lpd=false

# 种子交换, PT需要禁用, 默认:true
enable-peer-exchange=false

# 每个种子限速, 对少种的PT很有用, 默认:50K
#bt-request-peer-speed-limit=50K

# 设置 peer id 前缀
peer-id-prefix=-TR2770-

# 当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0
seed-ratio=0

# 强制保存会话, 即使任务已经完成, 默认:false
# 较新的版本开启后会在任务完成后依然保留.aria2文件
#force-save=false

# BT校验相关, 默认:true
#bt-hash-check-seed=true

# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=true

# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
bt-save-metadata=true

bt-max-open-files=16

# Http/FTP 相关
connect-timeout=120
```

3、启动aria2

```
aria2c --conf-path=/home/***/.aria2/aria2.conf
```

如果没有提示错误，按ctrl+c停止运行命令，转为后台运行：

```
aria2c --conf-path=/home/***/.aria2/aria2.conf -D
```

## 参数使用

1、使用 aria2 下载文件，只需在命令后附加地址即可。注意：命令是**aria2c**。比如我们下载ubuntu如：

```ruby
aria2c http://www.mirror.tw/pub/ubuntu/releases/jaunty/ubuntu-9.04-desktop-i386.iso
```



2、分段下载利用 aria2 的分段下载功能可以加快文件的下载速度，对于下载大文件时特别有用,为了**使用 aria2 的分段下载功能，你需要在命令中指定 s 选项**。如：

```ruby
aria2c -s 2 http://www.mirror.tw/pub/ubuntu/releases/jaunty/ubuntu-9.04-desktop-i386.iso
```



3、这将使用 2 个连接来下载该文件，s 后面的参数值介于 1~5 之间，你可以根据实际情况选择。
**使用 c 选项可以断点续传文件**如：

```ruby
aria2c -c http://www.mirror.tw/pub/ubuntu/releases/jaunty/ubuntu-9.04-desktop-i386.iso
```



4、下载 torrent 文件你也可以使用 aria2 下载 BitTorrent 文件。如：

```cpp
 aria2c -o gutsy.torrent http://cdimage.ubuntu.com/daily-live/current/gutsy-desktop-i386.iso.torrent
```

## 服务模式开机启动

编辑自启脚本

```kotlin
sudo vim /etc/init.d/aria2c
```

内容如下

```shell
#!/bin/sh
### BEGIN INIT INFO
# Provides: aria2
# Required-Start: $remote_fs $network
# Required-Stop: $remote_fs $network
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Aria2 Downloader
### END INIT INFO
 
case "$1" in
start)
 
 echo -n "已开启Aria2c"
 sudo aria2c --conf-path=/home/***/.aria2/aria2.conf -D
;;
stop)
 
 echo -n "已关闭Aria2c"
 killall aria2c
;;
restart)
 
 killall aria2c
 sudo aria2c --conf-path=/home/***/.aria2/aria2.conf -D
;;
esac
exit
```

2、修改文件权限为755 (a+x)

```kotlin
sudo chmod 755 /etc/init.d/aria2c
```



3、添加aria2c服务到开机启动

```shell
sudo update-rc.d aria2c defaults
```



4、启动服务

```shell
sudo service aria2c start
```



5、查看服务状态

```shell
sudo systemctl status aria2c
```