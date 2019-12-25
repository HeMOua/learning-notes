# curl使用

## 参数

+ `curl -s url`：静默模式，不输出任何东西
+ `curl -x proxy link`：设置代理，-x 参数等同于 --proxy

## 配置

每次使用`curl`的时候都会使用代理

```bash
# 修改curl配置文件
vim ~/.curlrc
# 写入
socks5 = "127.0.0.1:1024"

# 如果临时不需要代理使用以下参数
curl --noproxy "*" http://www.google.com
```



