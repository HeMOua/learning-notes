# 理解getResource()

## 案例

```java
public class TestClassLoader {
    public static void main(String[] args) {
        System.out.println(TestClassLoader.class.getResource(""));
        System.out.println(TestClassLoader.class.getResource("/"));
        System.out.println();
        System.out.println(TestClassLoader.class.getClassLoader().getResource(""));
        System.out.println(TestClassLoader.class.getClassLoader().getResource("/"));
    }
}
```

能得到结果

```out
file:/C:/myroad/utalitityUtils/target/classes/com/zsk/java/
file:/C:/myroad/utalitityUtils/target/classes/

file:/C:/myroad/utalitityUtils/target/classes/
null
```

那这两种方式有什么区别呢？下面跟源码一探究竟。

## 源码分析

1、首先分析`TestClassLoader.class.getResource("ehcache.xml")`

来看`Class`的`getResource`方法

```java
public java.net.URL getResource(String name) {
    name = resolveName(name);
    // 获取加载该Class的ClassLoader，sun.misc.Launcher$AppClassLoader@18b4aac2
    ClassLoader cl = getClassLoader0();
    if (cl==null) { //如果加载该Class的ClassLoader为null，则表示这是一个系统class
        // A system class.
        return ClassLoader.getSystemResource(name); //如果是系统class
    }
    return cl.getResource(name);//调用ClassLoader的getResource方法
}
```

接着看`ClassLoader`的`getResource`方法

```java
public URL getResource(String name) {
    URL url;
    if (parent != null) {//这里的parent为sun.misc.Launcher$ExtClassLoader@7d4793a8
        url = parent.getResource(name);//这里是一个递归调用，再次进入之后parent为null
    } else {
        url = getBootstrapResource(name);//到达系统启动类加载器
    }
    if (url == null) {//系统启动类加载器没有加载到，递归回退到第一次调用然后是扩展类加载器
        url = findResource(name);
    }
    return url;//最后如果都没有加载到，双亲委派加载失败，则加载应用本身自己的加载器。
}
```

关于上面的

+ sun.misc.Launcher$AppClassLoader@18b4aac2
+ sun.misc.Launcher$ExtClassLoader@7d4793a8

后面原理会详细介绍，源码没看明白，也许看了下面的原理 就豁然开朗了。

可以发现其实，**Class.getResource和ClassLoader.getResource 最终调用的是ClassLoader 类的getResource方法**。只不过Class.getResource是先调用Class 的 getResource 方法，在这个getResource  方法中，再去调用ClassLoader 类的getResource方法。

那么Class类中的getResource方法做了什么呢，主要的一句是 `name = resolveName(name); `

我们看一下这个代码实现：

```java
private String resolveName(String name) {
    if (name == null) {
        return name;
    }
    if (!name.startsWith("/")) {  //对于不以/开头的文件，
        Class<?> c = this;   //获取当前加载类的完整的类路径，我这里是com.zsk.java.TestClassLoader
        while (c.isArray()) {
            c = c.getComponentType();
        }
        String baseName = c.getName();
        int index = baseName.lastIndexOf('.');//找到文件的包名称
        if (index != -1) {
            name = baseName.substring(0, index).replace('.', '/')
                +"/"+name;//将包名称中的.替换为/ 并在最后加上/ 文件名
        }
    } else {
        name = name.substring(1);  //对于/开头的文件名，会只保留文件名称部分。
    }
    return name;
}
```

## 参考

https://blog.csdn.net/zhangshk_/article/details/82704010  部分说明有误