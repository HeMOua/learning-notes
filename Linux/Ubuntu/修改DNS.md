# 修改DNS

## 方法一

1、修改

```shell
sudo vim /etc/systemd/resolved.conf
```

2、

```conf
[Resolve]
DNS=114.114.114.114
DNS=8.8.8.8
#FallbackDNS=
#Domains=
#LLMNR=no
#MulticastDNS=no
#DNSSEC=no
#Cache=yes
#DNSStubListener=yes
```

3、查看结果

```
$ systemd-resolve --status
Global
DNS Servers: 114.114.114.114
8.8.8.8
DNSSEC NTA: 10.in-addr.arpa
16.172.in-addr.arpa
168.192.in-addr.arpa
17.172.in-addr.arpa
18.172.in-addr.arpa
19.172.in-addr.arpa
```

## 方法二

修改文件 **/etc/resolvconf/resolv.conf.d/base**（这个文件默认为空），添加以下内容：

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

:wq 保存退出

执行更新：

```
resolvconf -u
```

通过 **/etc/resolv.conf** 查看 DNS 设置:

可看到多了:

```
nameserver 8.8.8.8
nameserver  8.8.4.4
```

