# 利用Json序列化方式克隆

## 1、引入依赖

```XML
<!--jackson工具-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.12.3</version>
</dependency>
```

## 2、简单介绍

`ObjectMapper`可以从字符串、流或文件解析JSON，创建出Java对象。也可以将Java对象序列化为JSON

### 序列化

+ `writeValue()`：可以将序列化后的对象写入到指定文件、流中

+ `writeValueAsString()`：将序列化后的对象写入字符串
+ `writeValueAsBytes()`：将序列化后的对象写入字节数组

### 反序列化

移除了`getter/setter`方法名中的`get/set`字符部分, 并把方法名剩余字符的第一个字符小写, 得到的就是JSON属性名

如果想用其他方法来匹配JSON对象属性和Java对象属性, 可以自定义序列化/反序列化过程, 或者使用其他的**Jackson注解**

+ `readValue()`：从流、文件、字符串中反序列化json

## 3、克隆

```java
/**
     * 通过JSON序列化方式进行克隆
     *
     * @param entity        实例对象
     * @param copy          是否复制被忽略的属性
     * @param typeReference 返回类型
     * @return 克隆后的实体对象
     */
public static <E, T extends E> E 
    cloneByJson(E entity, TypeReference<T> typeReference, boolean copy) {
    if (entity == null) {
        return null;
    }
    ObjectMapper objectMapper = new ObjectMapper();
    try {
        // 进行序列化
        String json = objectMapper.writeValueAsString(entity);
        // 进行反序列化
        E result;
        if (typeReference == null) {
            result = (E) objectMapper.readValue(json, entity.getClass());
        } else {
            result = objectMapper.readValue(json, typeReference);
        }
        // 复制被忽略的属性
        if (copy) {
            copyIgnoreProp(entity, result);
            // 针对TableInfo对象做特殊处理
            if (entity instanceof TableInfo) {
                handlerTableInfo((TableInfo) entity, (TableInfo) result);
            }
        }
        return result;
    } catch (IOException e) {
        ExceptionUtil.rethrow(e);
    }
    return null;
}

/**
     * 复制属性
     *
     * @param oldEntity 就实体
     * @param newEntity 新实例
     */
private static void copyIgnoreProp(Object oldEntity, Object newEntity) {
    // 类型不一样直接返回
    if (!Objects.equals(oldEntity.getClass(), newEntity.getClass())) {
        return;
    }
    // 获取所有字段
    List<Field> fieldList = Arrays.asList(oldEntity.getClass().getDeclaredFields());
    if (fieldList.isEmpty()) {
        return;
    }
    fieldList.forEach(field -> {
        if (field.getAnnotation(JsonIgnore.class) != null) {
            // 设置允许访问
            field.setAccessible(true);
            // 复制字段
            try {
                field.set(newEntity, field.get(oldEntity));
            } catch (IllegalAccessException e) {
                throw new RuntimeException(e);
            }
        }
    });
}
```

稍微分析一下流程

1. 将对象序列化为JSON字符串
2. 从字符串中反序列化成java对象
3. 复制被忽略的属性

这个方法与js中的克隆挺像的，就是多了一个复制忽略属性