# EhCache 基础教程

## 开始

引入依赖

```xml
<dependency>
  <groupId>net.sf.ehcache</groupId>
  <artifactId>ehcache</artifactId>
  <version>2.10.3</version>
</dependency>  
```

在根目录创建文件 `ehcache.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="ehcache.xsd">
     <defaultCache
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="3600"
        timeToLiveSeconds="0"
        overflowToDisk="false"
        diskPersistent="false"
        diskExpiryThreadIntervalSeconds="120" />

    <cache
        name="student"
        maxEntriesLocalHeap="2000"
        eternal="false"
        timeToIdleSeconds="3600"
        timeToLiveSeconds="0"
        overflowToDisk="false"
        statistics="true">
    </cache>
</ehcache>
```

cache元素的属性：

+ `name`：缓存名称
+ `maxElementsInMemory`：内存中最大缓存对象数
+ `maxElementsOnDisk`：硬盘中最大缓存对象数，若是0表示无穷大
+ `eternal`：true表示对象永不过期，此时会忽略 `timeToIdleSeconds` 和`timeToLiveSeconds` 属性，默认为false
+ `overflowToDisk`：true表示当内存缓存的对象数目达到了maxELementsInMemory界限后，会把溢出的对象写到硬盘缓存中。注意：如果缓存的对象要写入到硬盘中的话，则该对象必须实现了Serializable接口才行。
+ `diskSpoolBufferSizeMB`：磁盘缓存区大小，默认为30MB。每个Cache都应该有自己的一个缓存区。
+ `diskPersistent`：是否缓存虚拟机重启期数据
+ `diskExpiryThreadIntervalSeconds`：磁盘失效线程运行时间间隔，默认为I20秒
+ `timeToIdleSeconds`：设定允许对象处于空闲状态的最长时间，以秒为单位。
  当对象自从最近一次被访问后，如果处于空闲状态的时间超过了 `timeToIdleSeconds` 属性值，这个对象就会过期，EHCache将把它从缓存中清空。只有当eternal属性为false,该属性才有效。如果该属性值为0，则表示对象可以无限期地处于空闲状态
+ `timeToLiveSeconds`：设定对象允许存在于缓存中的最长时间，以秒为单位。当对象自从被存放到缓存中后，如果处于缓存中的时间超过了timeToLiveSeconds属性值，这个对象就会过期，EHCache将把它从缓存中清除。只有当eternal属性为false,该属性才有效。
+ `memoryStoreEvictionPolicy`：当达到 maxELementsInMemory 限制时，Ehcache将会根据指定的策略去清理内存。
  + 可选策略有：LRU(最近最少使用，默认策略)、FIF0（先进先出）、LFU（最少访问次数）。

## 注解（整合Spring）

### `@Cacheable`

+ value：缓存位置名称，不能为空，如果使用EHCache，就是 ehcache.xml 中声明的 cache 的name，指明将值缓存到哪个
+ key：缓存值的键，可以通过键获取缓存的数据

为了保证 key 的唯一性，我们可以通过参数拼接的方式

```java
@Cacheable(value="UserCache", key="'user:' + #userId")
public void findById(Long userId);
```

+ keyGenerator：key生成器的bean名

当然也可以使用 key 生成器

```java
public static class SelfKeyGenerate implements KeyGenerator {
    @Override
    public Object generate(Object rage, Method method, Object... params) {
        return target.getClass().getSimpleName() + "#" method.getName();
    }
}
```

+ condition：当满足条件时，才进行缓存

```java
@Cacheable(value="UserCache", key="'user:' + #userId", condition="#userId.length() < 12")
public void findById(Long userId);
```

### `@CachePut`

与@Cacheable不同，@CachePut不仅会缓存方法的结果，还会执行方法的代码段。它支持的属性和用法都与@Cacheable一致。

### `@CacheEvict`

与@Cacheable功能相反，@CacheEvict表明所修饰的方法是用来删除失效或无用的缓存数据。

@CacheEvict支持如下几个参数：

+ value：缓存位置名称，不能为空，同上
+ key：缓存的key，默认为空，同上
+ condition：触发条件，只有满足条件的情况才会清除缓存，默认为空，支持SpEL
+ allEntries：true表示清除value中的全部缓存，默认为false

## 整合SpringBoot

**依赖**

```xml
<!-- 缓存starter -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>${spring-boot-cache.version}</version>
</dependency>

<!-- EhCache -->
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>${ehcache.version}</version>
</dependency>
```

**开启缓存**

在启动类上添加`@EnableCaching`