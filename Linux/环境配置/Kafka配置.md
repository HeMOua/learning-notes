1、下载 `kafka_2.13-2.8.0` 或其他版本

2、解压到目录 `tar -xzvf kafka-xxx`

3、设置自启

```
sudo vi /etc/systemd/system/kafka.service
```



```ini
[Unit]
Description=Kafka Service
After=network.target
[Service]
EnvironmentFile=/etc/environment
Type=forking
ExecStart=bash bin/kafka-server-start.sh -daemon config/server.properties
ExecReload=bash bin/kafka-server-start.sh -daemon config/server.properties
ExecStop=bash bin/kafka-server-stop.sh -daemon config/server.properties
WorkingDirectory=/usr/local/kafka/kafka_2.13-2.7.2/

#文件路径名
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
```

