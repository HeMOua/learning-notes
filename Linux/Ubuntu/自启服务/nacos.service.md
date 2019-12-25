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

