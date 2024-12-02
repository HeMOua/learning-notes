# 安装 OpenSSL

## 下载包

[Downloads | Library (openssl-library.org)](https://openssl-library.org/source/)

```shell
$ wget https://www.openssl.org/source/openssl-1.1.1w.tar.gz
$ tar -zxf openssl-1.1.1w.tar.gz
```

想要编译 Python，不要选择过高的 OpenSSL（urllib3 依赖它），会存在问题，选择 1.1.1 足够

```shell
$ cd openssl-1.1.1w
$ ./config -Wl,-rpath=/usr/lib64 --prefix=/usr/local/openssl --openssldir=/usr/local/openssl --libdir=/usr/lib64 && make -j 4 && make install
$ ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
```

