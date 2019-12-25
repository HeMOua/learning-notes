# SSH配置

## 文件

### config文件

```
Host github.com
  Hostname ssh.github.com
  Port 443
  
Host ubuntu
  Hostname 192.168.56.89
  IdentityFile ~/.ssh/id_rsa_2048
```

+ `Host xxx`：xxx就是目标主机的代号
  + 可以直接使用代号代替ip地址进行ssh连接
+ `Hostname`：目标主机的真实域名或ip地址
+ `IdentityFile`：用于指定私钥的位置所在
  + 如果**私钥不是默认的id_rsa**，一定要在config添加一项配置指定私钥的路径，然后**IP使用代号**进行连接

### 密钥文件

默认情况下，用户的 SSH 密钥存储在其 `~/.ssh` 目录下

+ id_rsa：默认的密钥名
+ id_rsa.pub：默认的公钥名

## 密钥

### 生成

```
ssh-keygen
```

参数：

+ `-o`：如果设置了密码，使用 `-o`参数会以比默认格式更能抗暴力破解的格式保存私钥

### 安装密钥

**从服务端安装**：

键入以下命令，在服务器上安装公钥：

```
[root@host ~]$ cd .ssh
[root@host .ssh]$ cat id_rsa.pub >> authorized_keys
```

如此便完成了公钥的安装。为了确保连接成功，请保证以下文件权限正确：

```
[root@host .ssh]$ chmod 600 authorized_keys
[root@host .ssh]$ chmod 700 ~/.ssh
```

**从客户端发送**：

```
 ssh-copy-id -i .ssh/id_rsa.pub  username@ip_address
```

## 配置密钥登录

编辑 /etc/ssh/sshd_config 文件，进行如下设置：

```
RSAAuthentication yes
PubkeyAuthentication yes
```

另外，请留意 root 用户能否通过 SSH 登录：

```
PermitRootLogin yes
```

当你完成全部设置，并以密钥方式登录成功后，再禁用密码登录：

```
PasswordAuthentication no
```

最后，重启 SSH 服务：

```
[root@host .ssh]$ service sshd restart
```

## 连接时长

### 客户端

编辑本地的SSH配置文件：`~/.ssh/config`

在这个文件后面加上如下一行：

```
ServerAliveInterval 120
```

这一行代码会让你的电脑在使用SSH连接服务器时，每隔120秒给服务器发送一个“空包”，保持它们之间的连接。

### 服务端

打开SSH服务的配置文件：`/etc/ssh/sshd_config`

加上如下两行：

```
ClientAliveInterval 60
ClientAliveCountMax 10
```

第一行，表示每隔 60 秒向客户端发送一个“空包”，以保持于客户端的连接。

第二行，表示总共发送 10 次“空包”，之后断开它们之间的连接

然后重启ssh服务：

```
/etc/init.d/ssh restart
```

再打开新终端连接就可以了，或者使用下面这个命令

```
service sshd restart
```

