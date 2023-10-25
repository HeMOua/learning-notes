1、安装，参考【工具与中间件 -> Zookeeper -> 安装】

2、自启

`sudo vi /etc/systemd/system/zookeeper.service`

```ini
[Unit]
Description=Zookeeper Service
After=network.target
[Service]
Environment="JAVA_HOME=/usr/local/java/jdk1.8.0_371"
Type=forking
ExecStart=bash /home/chenchao/ServerTools/zookeeper/bin/zkServer.sh start
ExecReload=bash /home/chenchao/ServerTools/zookeeper/bin/zkServer.sh start
ExecStop=bash /home/chenchao/ServerTools/zookeeper/bin/zkServer.sh stop
WorkingDirectory=/home/chenchao/ServerTools/zookeeper/bin

#文件路径名
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
```

