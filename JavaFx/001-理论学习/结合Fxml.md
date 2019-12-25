# 结合Fxml

## 基本

+ `fx:id`：任意一个控件都可以添加这个属性，然后在controller类中可以添加一个相同的控件作为类属性，并在属性上标注`@FXML`，就可以通过类属性来控制控件了

+ `text`：一般作为button或者label的文本内容，也可以通过前缀`%`显示国际化文件中的文本，如下

  ```xml
  <MenuItem mnemonicParsing="false" text="%closeAllTab"/>
  ```

+ `onAction`：指向事件处理函数

  ```xml
  <MenuItem mnemonicParsing="false" onAction="#closeAllTabAction"/>
  ```

  

## HBox或VBox

当位于HBox或VBox这两个容器内的组件想要铺满剩余空间时，可以设置`VBox.vgrow="ALWAYS"`，如下

```xml
<VBox>
    <TextArea VBox.vgrow="ALWAYS" />
</VBox>
```

## Tab

+ TablePane
  + tabMaxWidth：tab标签名的最大长度，过长时会以`...`表示
  + tabClosingPolicy
    + ALL_TABS：每个tab都有关闭按钮
    + SELECTED_TAB：只有被选中的tab才有关闭按钮
    + UNAVAILABLE：不可关闭