# last登录日志

### 命令介绍

| 命令      | 日志文件         | 功能                        |
| --------- | ---------------- | --------------------------- |
| `last`    | /var/log/wtmp    | 所有成功登录/登出的历史记录 |
| `lastb`   | /var/log/btmp    | 登录失败尝试                |
| `lastlog` | /var/log/lastlog | 最近登录记录                |

清空日志

```shell
echo -n "" > my_file
```

或者

```shell
truncate -s 0 my_file
```

