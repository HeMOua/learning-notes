# Nginx安装

## apt 安装

使用以下命令更新软件包列表：

```shell
sudo apt-get update
```


安装NGINX：

```shell
sudo apt-get install nginx
```

启动NGINX服务：

```shell
sudo systemctl start nginx
```

验证NGINX是否成功安装：

```shell
systemctl status nginx
```



主要的配置文件解释：

- 配置文件存放目录：`/etc/nginx`
- **主配置文件**：`/etc/nginx/conf/nginx.conf`
- 管理脚本：`/usr/lib64/systemd/system/nginx.service`
- 模块：`/usr/lisb64/nginx/modules`
- 应用程序：`/usr/sbin/nginx`
- 程序默认存放位置：`/usr/share/nginx/html`
- 日志默认存放位置：`/var/log/nginx`



## 编译安装

### 依赖

```shell
apt-get install gcc
apt-get install libpcre3 libpcre3-dev
apt-get install zlib1g zlib1g-dev
# Ubuntu14.04的仓库中没有发现openssl-dev，由下面openssl和libssl-dev替代
#apt-get install openssl openssl-dev
sudo apt-get install openssl 
sudo apt-get install libssl-dev
```

### 下载解压

```shell
wget http://nginx.org/download/nginx-1.13.7.tar.gz
tar -xvf nginx-1.13.7.tar.gz 
```

### 编译安装

```shell
# 进入nginx目录
cd xxx
# 执行命令
./configure
# 执行make命令
make
# 执行make install命令
make install
```

