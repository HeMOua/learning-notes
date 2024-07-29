[voidint/g: Golang Version Manager (github.com)](https://github.com/voidint/g)

0. 设置环境变量

   + `G_HOME="[gvm的根目录]"`

   + `G_EXPERIMENTAL=true`

1. 创建文件夹 `G_HOME\bin`，gvm 的根目录
2. 添加环境变量
   + `GOROOT=[gvm的根目录]\go`
   + Path 添加 `GOROOT\bin`、`G_HOME\bin`