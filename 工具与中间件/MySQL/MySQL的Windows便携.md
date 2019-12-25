## 下载

[MySQL :: Download MySQL Community Server![img](img/favicon.x-icon)https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)

## 解压，并创建`my.ini`文件

```ini
[mysql]
default-character-set=utf8

[mysqld]
port=3306
# 按需更改
basedir=E:/ServerTools/phpstudy_pro/Extensions/MySQL5.7.26/
# 设置 mysql数据库的数据的存放目录，MySQL 8+ 不需要以下配置，系统自己生成即可，否则有可能报错
datadir=E:/ServerTools/phpstudy_pro/Extensions/MySQL5.7.26/data/
character-set-server=utf8
# 或者 INNODB
default-storage-engine=MyIsam
max_connections=100
collation-server=utf8_unicode_ci
init_connect='SET NAMES utf8'
innodb_buffer_pool_size=64M
innodb_flush_log_at_trx_commit=1
innodb_lock_wait_timeout=120
innodb_log_buffer_size=4M
innodb_log_file_size=256M
interactive_timeout=120
join_buffer_size=2M
key_buffer_size=32M
log_error_verbosity=1
max_allowed_packet=16M
max_heap_table_size=64M
myisam_max_sort_file_size=64G
myisam_sort_buffer_size=32M
read_buffer_size=512kb
read_rnd_buffer_size=4M
server_id=1
skip-external-locking=on
sort_buffer_size=256kb
table_open_cache=256
thread_cache_size=16
tmp_table_size=64M
wait_timeout=120

[client]
port=3306
default-character-set=utf8
```

## 接下来我们来启动下 MySQL 数据库

以管理员身份打开 cmd 命令行工具，切换目录：

```
cd C:\web\mysql-8.0.11\bin
```

初始化数据库：

```
mysqld --initialize --console
```

执行完成后，会输出 root 用户的初始默认密码，如：

```
...
2018-04-20T02:35:05.464644Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: APWCY5ws&hjQ
...
```

**APWCY5ws&hjQ** 就是初始密码，后续登录需要用到，你也可以之后修改密码。

### 服务方式启动

输入以下安装命令：

```
mysqld install
```

启动输入以下命令即可：

```
net start mysql
```

> 注意: 在 5.7 需要初始化 data 目录：
>
> ```
> cd C:\web\mysql-8.0.11\bin 
> mysqld --initialize-insecure 
> ```
>
> 初始化后再运行 net start mysql 即可启动 mysql。

### 非服务方式启动

或者不以服务方式直接启动

```
mysqld --standalone
```



## 登录 MySQL

当 MySQL 服务已经运行时, 我们可以通过 MySQL 自带的客户端工具登录到 MySQL 数据库中, 首先打开命令提示符, 输入以下格式的命名:

```
mysql -h 主机名 -u 用户名 -p
```

参数说明：

- **-h** : 指定客户端所要登录的 MySQL 主机名, 登录本机(localhost 或 127.0.0.1)该参数可以省略;
- **-u** : 登录的用户名;
- **-p** : 告诉服务器将会使用一个密码来登录, 如果所要登录的用户名密码为空, 可以忽略此选项。

如果我们要登录本机的 MySQL 数据库，只需要输入以下命令即可：

```
mysql -u root -p
```

按回车确认, 如果安装正确且 MySQL 正在运行, 会得到以下响应:

```
Enter password:
```

若密码存在, 输入密码登录, 不存在则直接按回车登录。登录成功后你将会看到 Welcome to the MySQL monitor... 的提示语。

然后命令提示符会一直以 **mysql>** 加一个闪烁的光标等待命令的输入, 输入 **exit** 或 **quit** 退出登录。

## 修改密码

```
set password for root@localhost = password('root');  # mysql5.x
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';  # mysql8.x
```

