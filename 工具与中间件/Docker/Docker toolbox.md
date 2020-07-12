# Docker toolbox

## 排错

检查本地电脑的DNS，改为114.114.114.114

或者重启

## 换源

- 在windows命令行执行docker-machine ssh [machine-name]进入VM bash

- sudo vi /var/lib/boot2docker/profile

- 输入i进入编辑模式，输入esc退出编辑模式，输入:wq保存并退出。在--label provider=virtualbox的下一行添加--registry-mirror=https://xxxxxxxx.mirror.aliyuncs.com，这个地址可以在 [阿里云容器服务](https://cr.console.aliyun.com/cn-beijing/instances/mirrors) 获得。

- 重启docker服务：sudo /etc/init.d/docker restart 或者 重启VM：exit退出VM bash，在windows命令行中执行docker-machine restart

  