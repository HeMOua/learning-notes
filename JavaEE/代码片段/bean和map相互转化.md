# bean和map相互转化

```java
public class CommonUtil {

    /**
     * 把 map 转换成对象
     * @param map
     * @param clazz
     * @return 把Map转换成指定类型
     */
    @SuppressWarnings("rawtypes")
    public static <T> T toBean(Map map, Class<T> clazz) {
        try {
            /*
             * 1. 通过参数clazz创建实例
             * 2. 使用BeanUtils.populate把map的数据封闭到bean中
             */
            T bean = clazz.newInstance();
            ConvertUtils.register(new DateConverter(), java.util.Date.class);
            BeanUtils.populate(bean, map);
            return bean;
        } catch(Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static class DateConverter implements Converter {

        @SuppressWarnings("unchecked")
        @Override
        public Object convert(Class type, Object value) {
            if(value == null) return null;//如果要转换成值为null，那么直接返回null
            if(!(value instanceof String)) {//如果要转换的值不是String，那么就不转换了，直接返回
                return value;
            }
            String val = (String) value;//把值转换成String

            // 使用SimpleDateFormat进行转换
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            try {
                return sdf.parse(val);
            } catch (ParseException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```

