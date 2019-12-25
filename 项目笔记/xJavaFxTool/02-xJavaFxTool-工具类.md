# xJavaFxTool 工具类

## ConfigUtil

用来读取框架配置信息的，它含有的属性和枚举类型如下

```java
public class Config {

    /** 配置文件名 */
    public static final String CONFIG_FILE_NAME = "systemConfigure.properties";

    /** 设置系统语言 */
    public static Locale defaultLocale = Locale.getDefault();

    /** xJavaFxTool版本信息 */
    public static final String xJavaFxToolVersions = "V0.2.3";

    ///////////////////////////////////////////////////////////////

    public enum Keys {
        // 软件上次运行时的宽高和横纵坐标
        MainWindowWidth, MainWindowHeight, MainWindowTop, MainWindowLeft,
		// 语言环境
        Locale, 
        // 是否开启Notepad
        NotepadEnabled, 
        // 是否记住窗口位置
        RememberWindowLocation, 
        // 软件退出方法
        ConfirmExit,
        // 新启动页
        NewLauncher
    }

    /** 配置文件 */ 
    private static PropertiesConfiguration conf;
```

1、getConfig()

下面首先分析一下`getConfig()`方法

```java
public static PropertiesConfiguration getConfig() {
    try {
        if (conf == null) {
            File file = ConfigureUtil.getConfigureFile(CONFIG_FILE_NAME);
            conf = new PropertiesConfiguration(file);
            conf.setAutoSave(true); // 启用自动保存
        } else {
            conf.reload();
        }
    } catch (Exception e) {
        e.printStackTrace();

        // 即使加载失败，也要返回一个内存中的 PropertiesConfiguration 对象，以免程序报错。
        conf = new PropertiesConfiguration();
    }

    return conf;
}
```

因为存在一个未知的类`ConfigureUtil`，那么先将他解决吧

```java
public class ConfigureUtil {

    public static String getConfigurePath() {
        return System.getProperty("user.home") + "/xJavaFxTool/";
    }

    public static String getConfigurePath(String fileName) {
        return getConfigurePath() + fileName;
    }

    public static File getConfigureFile(String fileName) {
        return new File(getConfigurePath(fileName));
    }
}
```

查看源码就能发现是十分普通的工具类，无参方法先通过`System.getProperty()`获取用户目录，以此作为根目录。而有参方法就是将无参方法获取的路径，拼接上传递过来的参数，这样就确认了配置文件的路径。

回看`getConfig()`能发现还用到了`PropertiesConfiguration`这个类，所以我们还需要导入以下依赖

```xml
<dependency>
    <groupId>commons-configuration</groupId>
    <artifactId>commons-configuration</artifactId>
    <version>1.10</version>
</dependency>
```

方法的整体逻辑如下

+ 若`conf`为null说明第一次读取配置文件，则要根据配置文件路径初始化`conf`，如果配置文件不存在就会报错
  + 如果程序报错就返回一个内存中的`PropertiesConfiguration`
+ 若`conf`不为null，则通过`reload()`重载`conf`

2、其他set/get方法

其他方法就较普通了

```java
public static void set(Keys key, Object value) {
    getConfig().setProperty(key.name(), value);
}

public static String get(Keys key, String def) {
    Object value = getConfig().getProperty(key.name());
    return value == null ? def : value.toString();
}

public static int getInt(Keys key, int def) {
    return Integer.parseInt(get(key, String.valueOf(def)));
}

public static double getDouble(Keys key, double def) {
    return Double.parseDouble(get(key, String.valueOf(def)));
}

public static long getLong(Keys key, long def) {
    return Long.parseLong(get(key, String.valueOf(def)));
}

public static boolean getBoolean(Keys key, boolean def) {
    return Boolean.parseBoolean(get(key, String.valueOf(def)));
}
```



