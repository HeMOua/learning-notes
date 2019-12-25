# Font 相关

## 字体失效

JavaFx 有时设置了字体或者字体样式但不起效果，这可能与操作系统区域设置有关。

在 JavaFx 中，比如对于中国区域，字体“隶书”能起作用，但是“LuShu”就没作用，但是把电脑改成美国情况就恰恰相反。

## Font 文字压缩

1. fontsubsetpack：只选择需要的文字
2. 通过网站获取字体文件的 svg 格式，然后使用如下代码

```java
SVGPath svgPath = new SVGPath();
svgPath.setContent("...");

Region region = new Region();
region.setStyle("-fx-background-color: ...");
region.setPrefSize(100, 100);
region.setShape(svgPath);
```



