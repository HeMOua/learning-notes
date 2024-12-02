# WSL 相关操作

## 修改 wsl 主机名

```shell
vim /etc/wsl.conf
```

```ini
[network]
hostname = debian
generateHosts = false
```

