# SSH及服务器配置

安装前最好执行`sudo apt update`

## SSH

1. `sudo apt install openssh-server`，如果安装提示有依赖未满足，可以使用`sudo apt purge xx`重新安装相应的包
2. `sudo vi /etc/ssh/ssh_config`
3. 去掉`PasswordAuthentication yes`前面的#号
4. 把`PermitRootLogin prohibit-password`改成`PermitRootLogin yes`，保存退出
5. 重启服务`sudo /etc/init.d/ssh restart`、启动服务`sudo /etc/init.d/ssh start`

[Ubuntu 安装和配置ssh教程](https://blog.csdn.net/future_ai/article/details/81701744)

## SSH 测试连通性

```shell
ssh -v username@hostname_or_ip
```



## 传输秘钥

```shell
ssh-copy-id [-h|-?|-f|-n|-s] [-i [identity_file]] [-p port] [-F alternative ssh_config file] [[-o <ssh -o options>] ...] [user@]hostname
        -f: force mode -- copy keys without trying to check if they are already installed
        -n: dry run    -- no keys are actually copied
        -s: use sftp   -- use sftp instead of executing remote-commands. 
        -h|-?: print this help
```

