# 将网址URL转为json数据

```js
export function getQueryJson(url) {
  let param = {}; // 存储最终JSON结果对象
  url.replace(/([^?&]+)=([^?&]+)/g, function(m, k, v) {
    param[k] = decodeURIComponent(v)//解析字符为中文
  })
  return param
}
```

