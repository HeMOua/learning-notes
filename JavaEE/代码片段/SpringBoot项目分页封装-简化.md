# SpringBoot项目快速分页

## 1、先准备好工具类

```java
/**
 * 客户端工具类
 */
public class ServletUtils {

    /**
     * 获取请求属性
     *
     * @return 请求属性
     */
    public static ServletRequestAttributes getRequestAttributes() {
        RequestAttributes attributes = RequestContextHolder.getRequestAttributes();
        return (ServletRequestAttributes) attributes;
    }

    /**
     * 获取session
     */
    public static HttpSession getSession() {
        return getRequest().getSession();
    }

    /**
     * 获取request
     *
     * @return request
     */
    public static HttpServletRequest getRequest() {
        return getRequestAttributes().getRequest();
    }

    /**
     * 获取response
     *
     * @return response
     */
    public static HttpServletResponse getResponse() {
        return getRequestAttributes().getResponse();
    }

    /**
     * 获取String参数
     */
    public static String getParameter(String name) {
        return getRequest().getParameter(name);
    }

    /**
     * 获取String参数
     */
    public static String getParameter(String name, String defaultValue) {
        return Convert.toStr(getRequest().getParameter(name), defaultValue);
    }


    /**
     * 获取Integer参数
     */
    public static Integer getParameterToInt(String name) {
        return Convert.toInt(getRequest().getParameter(name));
    }

    /**
     * 获取Integer参数
     */
    public static Integer getParameterToInt(String name, Integer defaultValue) {
        return Convert.toInt(getRequest().getParameter(name), defaultValue);
    }
}
```

## 2、分页参数以及响应结果常量

```java
public class Constant {

    /**
     * 操作成功
     */
    public static final int SUCCESS = "200";

    /**
     * 系统内部错误
     */
    public static final int ERROR = "500";

    /**
     * 当前记录起始索引
     */
    public static final String PAGE_NUM = "pageNum";

    /**
     * 每页显示记录数
     */
    public static final String PAGE_SIZE = "pageSize";
}
```

## 3、封装分页数据类

```java
public class TableDataInfo {

    /** 总记录数 */
    private long total;

    /** 列表数据 */
    private List<?> rows;

    /** 消息状态码 */
    private int code;

    /** 消息内容 */
    private int msg;

    public TableDataInfo() {
    }

    public TableDataInfo(List<?> rows, int total) {
        this.total = total;
        this.rows = rows;
    }
	。。。getter setter
}
```

```java
public class RespResult extends HashMap<String, Object> {

    public static final String CODE_TAG = "code";
    public static final String MSG_TAG = "msg";
    public static final String DATA_TAG = "data";

    public RespResult(int code, String msg, Object data) {
        put(CODE_TAG, code);
        put(MSG_TAG, msg);
        put(DATA_TAG, data);
    }

    public static RespResult success(String msg, Object data) {
        return new RespResult(Constant.SUCCESS, msg, data);
    }

    public static RespResult success(Object data) {
        return success("操作成功", data);
    }

    public static RespResult success() {
        return success("操作成功", null);
    }

    public static RespResult error(String msg, Object data) {
        return new RespResult(Constant.ERROR, msg, data);
    }

    public static RespResult error(String msg) {
        return error(msg, null);
    }

    public static RespResult error() {
        return error("操作失败");
    }
}
```

## 4、添加pageHelper依赖并配置

pom.xml

```xml
<!--mybatis-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.0</version>
</dependency>
<!--分页插件-->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>
```

application.yaml

1. 第一步端口号： `server.port`
2. 第二步服务名：`spring.application.name`
3. 第三步数据源：`spring.datasource`
4. 第四步mybatis xml文件位置配置、alias包配置：`mybatis`

```yaml
server:
  port: 8080
spring:
  application:
    name: music-web
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/music?characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: root
mybatis:
  mapper-locations: classpath:mapper/*Dao.xml
  type-aliases-package: com.music.project.domain
```

## 5、BaseController

```java
public class BaseController {

    // 开始分页
    protected void startPage() {
        Integer pageNum = ServletUtils.getParameterToInt(Constant.PAGE_NUM);
        Integer pageSize = ServletUtils.getParameterToInt(Constant.PAGE_SIZE);
        if (ObjectUtil.isNotNull(pageNum) && ObjectUtil.isNotNull(pageSize)) {
            PageHelper.startPage(pageNum, pageSize);
        }
    }

    // 返回分页数据
    protected TableDataInfo getTableData(List<?> list) {
        TableDataInfo tableData = new TableDataInfo();
        tableData.setCode(Constant.SUCCESS);
        tableData.setRows(list);
        tableData.setTotal(new PageInfo<>(list).getTotal());
        return tableData;
    }

    /**
     * 响应返回结果
     *
     * @param rows 影响行数
     * @return 操作结果
     */
    protected RespResult toResult(int rows) {
        return rows > 0 ? RespResult.success() : RespResult.error();
    }
}
```

