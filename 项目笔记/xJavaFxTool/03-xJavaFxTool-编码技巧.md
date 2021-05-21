# 编码技巧

## 程序流程

1、加载国际化文件

2、通过`FxApp`工具类初始化页面参数、图标和样式文件

## 通过XML创建Scene

1、新建一个`FXMLLoader`类

2、设置`xmlLoader`的location，也就是xml所在的位置

3、如果配置了国际化，需要设置一下`resources`

4、最后通过`xmlLoader`的`load`方法可以获取一个Paraent组件，然后调用构造方法`new Scene()`，将获取的组件作为构造方法参数就可以获取一个`Scene`了，例子如下

```java
FXMLLoader fxmlLoader = new FXMLLoader();
fxmlLoader.setLocation(Main.class.getResource("/fxmlView/main.fxml"));
fxmlLoader.setResources(RESOURCE_BUNDLE);

Parent root = fxmlLoader.load();
primaryStage.setScene(new Scene(root));
```

