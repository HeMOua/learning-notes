## Could not validate integrity of download from http://0.0.0.0/...

Maven到3.8.1后，mvn编译的时候总是提示拉不到依赖，因为从 3.8.1 开始就要求必须使用 https 协议，对于 http 协议的仓库，就会报错

+ 解决办法一：
  + 降低 maven 版本到 3.8.1 之后，有时 idea 反应不过来，需要重启一下
  + 