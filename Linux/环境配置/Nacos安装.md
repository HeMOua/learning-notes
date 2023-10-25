1、安装，可以参考 【JavaEE -> SpringCloud -> Nacos】或者 【项目笔记 -> 谷粒商城 -> 注册中心...】

2、自启

`vim /etc/systemd/system/nacos.service`

```ini
[Unit]
Description=Nacos Service
After=network.target
[Service]
Environment="JAVA_HOME=/usr/local/java/jdk1.8.0_371"
Type=forking
ExecStart=bash /home/chenchao/ServerTools/nacos/bin/startup.sh -m standalone
ExecReload=bash /home/chenchao/ServerTools/nacos/bin/shutdown.sh
ExecStop=bash /home/chenchao/ServerTools/nacos/bin/shutdown.sh
WorkingDirectory=/home/chenchao/ServerTools/nacos/bin

#文件路径名
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
```

