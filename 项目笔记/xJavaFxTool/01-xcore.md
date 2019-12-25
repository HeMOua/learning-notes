# xcore

## helper 包

在此包内，定义一些帮助性的静态方法，用来构建各种组件

### ImageHelper

用来创建一些Image、ImageView组件，也可以算是工具类了

```java
public class ImageHelper {

    public static Image icon(String resourcePath) {
        URL resource = ImageHelper.class.getResource(resourcePath);
        if (resource == null) {
            throw new ResourceException("Resource '" + resourcePath + "' not found.");
        } else {
            return new Image(resource.toExternalForm());
        }
    }

    public static ImageView iconView(Image icon) {
        return new ImageView(icon);
    }

    public static ImageView iconView(String resourcePath) {
        return iconView(icon(resourcePath));
    }

    public static ImageView iconView(Image icon, double size) {
        ImageView imageView = iconView(icon);
        imageView.setFitHeight(size);
        imageView.setFitWidth(size);
        return imageView;
    }

    public static ImageView iconView(String resourcePath, double size) {
        return iconView(icon(resourcePath), size);
    }
}
```

## Util 包

### JavaFxSystemUtil

初始化或保存软件配置，比如jar包和国际化配置

```java
@Slf4j
public class JavaFxSystemUtil {

    /**
     * 初始化本地语言
     */
    public static void initSystemLocal() {
        try {
            String localeString = Config.get(Config.Keys.Locale, "");

            if (StringUtils.isNotEmpty(localeString)) {
                String[] locale1 = localeString.split("_");
                Config.defaultLocale = new Locale(locale1[0], locale1[1]);
            }

            App.RESOURCE_BUNDLE = ResourceBundle.getBundle("locale.Menu", Config.defaultLocale);
        } catch (Exception e) {
            log.error("初始化本地语言失败", e);
        }
    }
    
    /**
     * 保存Stage边框位置
     */
    public static void savePrimaryStageBound(Stage stage) {
        if (!Config.getBoolean(Keys.RememberWindowLocation, true)) {
            return;
        }
        if (stage == null || stage.isIconified()) {
            return;
        }
        try {
            Config.set(Keys.MainWindowLeft, stage.getX());
            Config.set(Keys.MainWindowTop, stage.getY());
            Config.set(Keys.MainWindowWidth, stage.getWidth());
            Config.set(Keys.MainWindowHeight, stage.getHeight());
        } catch (Exception e) {
            log.error("初始化界面位置失败：", e);
        }
    }
    
    /**
     * 加载Stage边框位置
     */
    public static void loadPrimaryStageBound(Stage stage) {
        try {
            if (!Config.getBoolean(Keys.RememberWindowLocation, true)) {
                return;
            }

            double left = Config.getDouble(Keys.MainWindowLeft, -1);
            double top = Config.getDouble(Keys.MainWindowTop, -1);
            double width = Config.getDouble(Keys.MainWindowWidth, -1);
            double height = Config.getDouble(Keys.MainWindowHeight, -1);

            if (left > 0) {
                stage.setX(left);
            }
            if (top > 0) {
                stage.setY(top);
            }
            if (width > 0) {
                stage.setWidth(width);
            }
            if (height > 0) {
                stage.setHeight(height);
            }
        } catch (Exception e) {
            log.error("初始化界面位置失败：", e);
        }
    }
}
```

+ `initSystemLocal`
  + 通过`Config`先从配置文件中获取本地语言，如不存在配置返回空
  + 如果本地语言配置不为空，则设置Config内的`defalutLocale`为配置值，否则不用管（Confg中defaultLocale初始值为`Locale.getDefault()`）
  + 最后通过`ResourceBundle.getBundle`方法获取国际化配置就行
+ `savePrimaryStageBound`
  + 首先通过Confg的中的配置判断是否要记住软件位置，如果不同则直接退出函数
  + 如果窗口不存在，或者最小化了，也直接退出函数
  + 通过`Confg`设置软件坐标和宽高

### ModalUtil

创建各种提示框

1、Alert 警告提示框

```java
public class ModalUtil {

    public static boolean confirmOkCancel(String title, String message) {
        return confirm(AlertType.CONFIRMATION, title, message, ButtonType.OK, ButtonType.CANCEL) == ButtonType.OK;
    }

    public static boolean confirmYesNo(String title, String message) {
        return confirm(AlertType.WARNING, title, message, ButtonType.YES, ButtonType.NO) == ButtonType.YES;
    }

    public static ButtonType confirmYesNoCancel(String title, String message) {
        return confirm(AlertType.WARNING, title, message, ButtonType.YES, ButtonType.NO, ButtonType.CANCEL);
    }

    public static ButtonType confirm(AlertType alertType, String title, String message, ButtonType... buttonTypes) {
        try {
            Alert alert = new Alert(alertType, message, buttonTypes);
            alert.setTitle(title);
            alert.setHeaderText((String)null);
            Stage stage = (Stage)alert.getDialogPane().getScene().getWindow();
            FxApp.setupIcon(stage);
            FxApp.setupModality(alert);
            Optional<ButtonType> result = alert.showAndWait();
            return (ButtonType)result.orElse(ButtonType.CANCEL);
        } catch (Exception e) {
            e.printStackTrace();
            return ButtonType.CANCEL;
        }
    }
}
```



## FxApp

用来记录当前Fx程序的stage、icon、style等，以及其他操作

```java
public class FxApp {

    public static Stage primaryStage;
    public static Image appIcon;
    public static final List<String> styleSheets = new ArrayList<>();

    public static void init(Stage primaryStage, String iconPath) {
        FxApp.primaryStage = primaryStage;
        if (StringUtils.isNotBlank(iconPath)) {
            appIcon = ImageHelper.icon(iconPath);
            primaryStage.getIcons().add(appIcon);
        }
    }

    public static void setupIcon(Stage stage) {
        if (appIcon != null) {
            stage.getIcons().clear();
            stage.getIcons().add(appIcon);
        }
    }

    public static void addStyleSheet(String stylePath) {
        URL url = FxApp.class.getResource(stylePath);
        if (url != null) styleSheets.add(url.toExternalForm());
    }

    public static void setupModality(Dialog<?> stage) {
        if (stage != null && primaryStage != null && primaryStage.isShowing()) {
            stage.initOwner(primaryStage);
            stage.initModality(Modality.WINDOW_MODAL);
        }
    }
}
```

+ `setupModality`
  + 只有在窗口存在且为显示状态并且模态窗口也存在时，才能给模态窗口设置模态模式