# Nginx安装

## 依赖

```shell
apt-get install gcc
apt-get install libpcre3 libpcre3-dev
apt-get install zlib1g zlib1g-dev
# Ubuntu14.04的仓库中没有发现openssl-dev，由下面openssl和libssl-dev替代
#apt-get install openssl openssl-dev
sudo apt-get install openssl 
sudo apt-get install libssl-dev
```

## 下载解压

```shell
wget http://nginx.org/download/nginx-1.13.7.tar.gz
tar -xvf nginx-1.13.7.tar.gz 
```

## 编译安装

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

