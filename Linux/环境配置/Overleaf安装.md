# Overleaf安装

## 基本环境配置

安装Docker：

```javascript
sudo curl -sSL https://get.daocloud.io/docker | sh
```

查看版本，检查是否安装成功：

```javascript
docker --version
docker-compose --version
```

启动Docker：

```javascript
systemctl start docker
```

## 运行容器

### 安装overleaf

首先将overleaf项目从github拉至本地

```javascript
git clone https://github.com/overleaf/toolkit.git ./overleaf
```

然后进行初始化配置

```javascript
cd ./overleaf
bin/init
```

这里我们进入config下的overleaf.rc文件进行配置：

```javascript
vim ./config/overleaf.rc
```

更改如下两行：

```javascript
SHARELATEX_LISTEN_IP=如果本地使用按照原配置即可，腾讯云服务器需要改为**内网地址**
SHARELATEX_PORT=想映射的端口，默认的80端口常常被占用
```

（注意，映射的端口需要在腾讯云防火墙和centos内部的防火墙中均被开放）

其中还有许多个性化的配置：如网页抬头文字内容，网址标题，UI语言（中文）等，可以在[overleaf的Wiki页面](https://link.zhihu.com/?target=https%3A//github.com/overleaf/overleaf/wiki)中进行查看和配置。

执行容器：`bin/up`。

此时正在拉取镜像，可以等出现大量的log时使用 ctrl+c 停止，然后执行`bin/start`即可。

此时用浏览器打开`http://公网IP:映射的端口/launchpad`应该能看到管理员注册界面，至此overleaf的安装结束。

## 下载完整texlive包

以上安装的overleaf配套的LaTeX不是完整版，所以需要继续下载。

首先进入容器的bash:

```javascript
docker exec -it sharelatex bash
cd /usr/local/texlive
```

然后执行以下命令：

```javascript
# 下载并运行升级脚本
wget http://mirror.ctan.org/systems/texlive/tlnet/update-tlmgr-latest.sh
sh update-tlmgr-latest.sh -- --upgrade

# 更换texlive的下载源
tlmgr option repository https://mirrors.sustech.edu.cn/CTAN/systems/texlive/tlnet/

# 升级tlmgr
tlmgr update --self --all

# 安装完整版texlive（时间比较长，不要让shell断开）
tlmgr install scheme-full

# 退出sharelatex的命令行界面，并重启sharelatex容器
exit
docker restart sharelatex
```

至此，安装完成。

## 配置反向代理【可选】

如果想要使用域名ssl访问，或者改变访问端口，可能需要反向代理。

**宝塔可能要修改反向配置文件为下面内容，如果直接默认的话，每次加载项目文档会卡卡的，甚至提示XXX加载错误**

```javascript
#PROXY-START/
location ~* \.(php|jsp|cgi|asp|aspx)$
{
    proxy_pass http://127.0.0.1:5200;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header REMOTE-HOST $remote_addr;
}
location /
{
    proxy_pass http://127.0.0.1:5200;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header REMOTE-HOST $remote_addr;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'Upgrade';
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_read_timeout 3m;
    proxy_send_timeout 3m;

    add_header X-Cache $upstream_cache_status;
    #Set Nginx Cache
    proxy_ignore_headers Set-Cookie Cache-Control expires;
    add_header Cache-Control no-cache;
    expires 12h;
}

#PROXY-END/
```

以上内容部分参考：

[服务器部署Overleaf](https://zlogs.net/blog/21/11101200/)

[利用腾讯云服务器搭建自己的overleaf（写论文神器）](https://zhuanlan.zhihu.com/p/481788258)