1、安装

下载 Nacos Server https://github.com/alibaba/nacos/releases

2、启动

+ Linux/Unix/Mac 操作系统，执行命令 `sh startup.sh -m standalone`
+ Windows 操作系统，执行命令 `cmd startup.cmd`

3、自启

`vim /etc/systemd/system/nacos.service`

```properties
[Unit]
Description=Nacos Service
After=network.target
[Service]
EnvironmentFile=/etc/environment
Type=forking
ExecStart=bash startup.sh -m standalone
ExecReload=bash shutdown.sh
ExecStop=bash shutdown.sh
WorkingDirectory=/usr/local/nacos/bin

#文件路径名
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
```

4、修改 `startup.sh`，文件好像有问题，手动添加环境变量配置

```
JAVA_HOME=/usr/local/java/jdk1.8.0_371
```

