# Alpine Linux

## 安装

https://blog.csdn.net/csdn_duomaomao/article/details/76053229

01-Alpine Linux光盘启动1


02-Alpine Linux光盘启动2


03-系统登录以后的提示信息-包括使用帮助网址 http://wiki.alpinelinux.org -系统安装命令 setup-alpine


04-安装Linux到硬盘命令 setup-alpine--键盘布局-主机名 alpine -网卡 eth0


05-eth0网卡设置帮助-IP地址设置-网关设置等，使用静态IP地址


06-网关-域名-root管理员用户口令修改


07-时区设置-Asia


08-子时区设置-Shanghai


09-软件仓库的地址-默认选 f，会自动检测最快的镜像站点


10-手工选择软件仓库的镜像地址 3 -选择安装openssh 服务器-NTP客户端-磁盘的选择 sda


11-本地磁盘的使用方式选择-


12-获取磁盘使用的帮助说明，sys 、data、 lvm 等简要解释


13-选择 sys 方式使用磁盘，将系统安装到本地硬盘-格式化硬盘-完成硬盘安装

14-重新启动Linux后的界面-机器名已经是刚才设置的名称了

【完成安装】

## 配置ssh便于xshell等客户端连接

`vi /etc/ssh/sshd_config`,将 permitrootlogin 设置为yes，`service sshd restart`就可以使用xshell连接。

