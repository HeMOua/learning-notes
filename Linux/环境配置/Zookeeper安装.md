1、安装，参考【工具与中间件 -> Zookeeper -> 安装】

2、自启

`sudo vi /etc/systemd/system/zookeeper.service`

```ini
[Unit]
Description=Zookeeper Service
After=network.target
[Service]
EnvironmentFile=/etc/environment
Type=forking
ExecStart=bash zkServer.sh start
ExecReload=bash zkServer.sh start
ExecStop=bash zkServer.sh stop
WorkingDirectory=/usr/local/zookeeper/zookeeper-3.4.11/bin

#文件路径名
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
```

