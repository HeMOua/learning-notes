# vsCode配置

## jsconfig.json

当我们配置了路径昵名时，编译器就不能对昵名开头的文件进行智能提示了，如下方法可解决

在项目根目录，说通俗点，就是和 package.json 同一个文件夹下，创建一个名为 `jsconfig.json` 的文件，内容如下

```js
{ 
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
        "@/*": ["src/*"]
    }
  },
  "exclude": ["node_modules", "dist"]
}
```

重点是 `paths` 的配置，它可以将 `@/`  开头的文件路径映射成我们想要的路径