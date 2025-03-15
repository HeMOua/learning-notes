# 《黑马架构师_源码系列-主流框架&中间件》-- MyBatis

```markdown
* 手写持久层框架-仿写mybatis
* Mybatis架构设计&主要组件
* Mybatis如何完成的初始化?
* Mybatis如何完成的sql解析及执行?
* Mybatis如何设置的参数?
* Mybatis如何进行的类型转换?
* Mybatis如何封装的返回结果集?
* Mybatis插件原理是什?
* Mybatis缓存底层数据结构是什么?
```

## 1. 手写持久层框架-ipersistent

### 1.1 JDBC操作数据库_问题分析

JDBC API 允许应用程序访问任何形式的表格数据，特别是存储在关系数据库中的数据

![image-20210812103601952](.\img\image-20210812103601952.png)



代码示例：

```java
public static void main(String[] args) { 
    	Connection connection = null;
		PreparedStatement preparedStatement = null;
		ResultSet resultSet = null;
	try {
	// 加载数据库驱动
	Class.forName("com.mysql.jdbc.Driver");
	// 通过驱动管理类获取数据库链接
	connection =
DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis? characterEncoding=utf-8", "root", "root");
	// 定义sql语句？表示占位符
	String sql = "select * from user where username = ?";
	// 获取预处理statement
	preparedStatement = connection.prepareStatement(sql);
	// 设置参数，第一个参数为sql语句中参数的序号(从1开始)，第二个参数为设置的参数值 preparedStatement.setString(1, "tom");
	// 向数据库发出sql执行查询，查询出结果集
	resultSet = preparedStatement.executeQuery();
	// 遍历查询结果集
	while (resultSet.next()) {
		int id = resultSet.getInt("id");
		String username = resultSet.getString("username");
	// 封装User
		user.setId(id);
		user.setUsername(username);
	}
		System.out.println(user);
	}
	} catch (Exception e) {
		e.printStackTrace();
	} finally {
		  // 释放资源
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (preparedStatement != null) {
            try {
                preparedStatement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
	}
}
```



### 1.2 JDBC问题分析&解决思路

剖开代码，逐个分析：

**（1）加载驱动，获取链接：**

<img src=".\img\image-20210811175322483.png" alt="image-20210811175322483" style="zoom:80%;" />

- 存在问题1：数据库配置信息存在**硬编码**问题。

  > 优化思路：使用配置文件！

- 存在问题2：频繁创建、释放**数据库连接**问题。

  > 优化思路：使用数据连接池！



**（2）定义sql、设置参数、执行查询：**

<img src=".\img\image-20210811175342646.png" alt="image-20210811175342646" style="zoom:80%;" />

- 存在问题3：SQL语句、设置参数、获取结果集参数均存在**硬编码**问题 。

  > 优化思路：使用配置文件！



**（2）遍历查询结果集：**

<img src=".\img\image-20210811175401307.png" alt="image-20210811175401307" style="zoom:80%;" />

- 存在问题4：**手动封装**返回结果集，较为繁琐

  > 优化思路：使用Java反射、内省！

针对JDBC各个环节中存在的不足，现在，我们整理出对应的优化思路，统一汇总：

| 存在问题                                          | 优化思路           |
| ------------------------------------------------- | ------------------ |
| 数据库配置信息存在硬编码问题                      | 使用配置文件       |
| 频繁创建、释放数据库连接问题                      | 使用数据连接池     |
| SQL语句、设置参数、获取结果集参数均存在硬编码问题 | 使用配置文件       |
| 手动封装返回结果集，较为繁琐                      | 使用Java反射、内省 |



### 1.3 自定义持久层框架_思路分析

JDBC是个人作战，凡事亲力亲为，低效而高险，自己加载驱动，自己建连接，自己 …

而持久层框架好比是多工种协作，分工明确，执行高效，有专门负责解析注册驱动建立连接的，有专门管理数据连接池的，有专门执行sql语句的，有专门做预处理参数的，有专门装配结果集的 …

>  优化思路： 框架的作用，就是为了帮助我们减去繁重开发细节与冗余代码，使我们能更加专注于业务应用开发。



#### 使用JDBC和使用持久层框架区别：

![](.\img\对比.jpg)

是不是发现，拥有这么一套持久层框架是如此舒适，我们仅仅需要干两件事：

- **配置数据源**（地址/数据名/用户名/密码）
- **编写SQL与参数准备**（SQL语句/参数类型/返回值类型）



##### 框架，除了思考本身的工程设计，还需要考虑到实际项目端的使用场景，干系方涉及两端：

- 使用端（实际项目）
- 持久层框架本身

以上两步，我们通过一张架构图《 **手写持久层框架基本思路** 》来梳理清楚：

<img src=".\img\3.jpg" style="zoom:75%;" />



#### 核心接口/类重点说明：

| 分工协作                                        | 角色定位          | 类名定义                 |
| ----------------------------------------------- | ----------------- | ------------------------ |
| 负责读取配置文件                                | 资源辅助类        | Resources                |
| 负责存储数据库连接信息                          | 数据库资源类      | Configuration            |
| 负责存储SQL映射定义、存储结果集映射定义         | SQL与结果集资源类 | MappedStatement          |
| 负责解析配置文件，创建会话工厂SqlSessionFactory | 会话工厂构建者    | SqlSessionFactoryBuilder |
| 负责创建会话SqlSession                          | 会话工厂          | SqlSessionFactory        |
| 指派执行器Executor                              | 会话              | SqlSession               |
| 负责执行SQL （配合指定资源Mapped Statement）    | 执行器            | Executor                 |

> 正常来说项目只对应一套数据库环境，一般对应一个SqlSessionFactory实例对象，我们使用单例模式只创建一个SqlSessionFactory实例。
>
> 如果需要配置多套数据库环境，那需要做一些拓展，例如Mybatis中通过environments等配置就可以支持多套测试/生产数据库环境进行切换。





#### **项目使用端：**

（1）调用框架API，除了引入自定义持久层框架的jar包

（2）提供两部分配置信息：1.sqlMapConfig.xml : 数据库配置信息（地址/数据名/用户名/密码），以及mapper.xml的全路径

​                                                2.mapper.xml : SQL配置信息，存放SQL语句、参数类型、返回值类型相关信息



#### **自定义框架本身：**

1、加载配置文件：根据配置文件的路径，加载配置文件成字节输入流，存储在内存中。

<img src=".\img\image-20210811180809813.png" alt="image-20210811180809813" style="zoom:80%;" />

2、 创建两个javaBean（容器对象）：存放配置文件解析出来的内容

<img src=".\img\image-20210811180923884.png" alt="image-20210811180923884" style="zoom:80%;" />

3、解析配置文件（使用dom4j） ，并创建SqlSession会话对象

<img src=".\img\image-20210811180948093.png" alt="image-20210811180948093" style="zoom:80%;" />

4、创建SqlSessionFactory接口以及实现类DefaultSqlSessionFactory

<img src=".\img\image-20210811181025728.png" alt="image-20210811181025728" style="zoom:80%;" />

5、创建SqlSession接口以及实现类DefaultSqlSession

<img src=".\img\image-20210811181045390.png" alt="image-20210811181045390" style="zoom:80%;" />

6、创建Executor接口以及实现类SimpleExecutor

<img src=".\img\image-20210811181106435.png" alt="image-20210811181106435" style="zoom:80%;" />



基本过程我们已经清晰，我们再细化一下类图，更好的助于我们实际编码：

![](.\img\4.jpg)



##### 最终手写的持久层框架结构参考：

<img src=".\img\image-20210811181425774.png" alt="image-20210811181425774" style="zoom:67%;" />



### 1.4 自定义持久层框架_编码

```xml
  <properties>
        <!-- Encoding -->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
        <java.version>11</java.version>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

  <!--引入ipersistent的依赖>
```

在使用端项目中创建配置配置文件

创建 sqlMapConfig.xml

```xml
<configuration> 
    <!--1.配置数据库配置信息-->
    <dataSource>
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql:///zdy_mybatis?useSSL=false&amp;characterEncoding=UTF-8&amp;serverTimezone=UTC"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </dataSource>


    <!--2.引入映射配置文件-->
    <mappers>
        <mapper resource="mapper/UserMapper.xml"></mapper>
    </mappers>

</configuration> 
```

mapper.xml

```xml
<mapper namespace="User">
    
    <!--根据条件查询单个-->
    <select id="selectOne" resultType="com.itheima.pojo.User" parameterType="com.itheima.pojo.User">
        select * from user where id = #{id} and username = #{username}
    </select>

	
  <!--查询所有-->
    <select id="selectList" resultType="com.itheima.pojo.User">
        select * from user
    </select>
</mapper>
```

User实体

```java
public class User {
	//主键标识
	private Integer id;
	//用户名
	private String username;
    
	public Integer getId() { 
        return id;
	}
	public void setId(Integer id) { 
        this.id = id;
	}
	public String getUsername() { 
        return username;
	}
	public void setUsername(String username) { 
        this.username = username;
	}

    @Override
	public String toString() {
		return "User{" +
		"id=" + id +
		", username='" + username + '\'' + '}';
	}
}
```

再创建一个Maven子工程并且导入需要用到的依赖坐标

```xml
  <properties>
        <!-- Encoding -->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
        <java.version>11</java.version>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- mysql 依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.6</version>
        </dependency>

        <!--junit 依赖-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <!--作用域测试范围-->
            <scope>test</scope>
        </dependency>

        <!--dom4j 依赖-->
        <dependency>
            <groupId>dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>1.6.1</version>
        </dependency>

        <!--xpath 依赖-->
        <dependency>
            <groupId>jaxen</groupId>
            <artifactId>jaxen</artifactId>
            <version>1.1.6</version>
        </dependency>


        <!--druid连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.21</version>
        </dependency>

        <!-- log日志 -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
       
    </dependencies>
```

Resources

```java
public class Resources {

    /**
     * 根据配置文件的路径，加载成字节输入流，存到内存中
     * @param path
     * @return
     */
    public static InputStream getResourceAsSteam(String path){
        InputStream resourceAsStream = Resources.class.getClassLoader().getResourceAsStream(path);
        return resourceAsStream;
    }
```



Configuration

```java
/**
 * 存放核心配置文件解析的内容
 */
public class Configuration {

    // 数据源对象
    private DataSource dataSource;

    // map : key :statementId  value : 封装好的MappedStatement
    private Map<String,MappedStatement> mappedStatementMap = new HashMap<>();

    public DataSource getDataSource() {
        return dataSource;
    }

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public Map<String, MappedStatement> getMappedStatementMap() {
        return mappedStatementMap;
    }

    public void setMappedStatementMap(Map<String, MappedStatement> mappedStatementMap) {
        this.mappedStatementMap = mappedStatementMap;
    }
}
```

MappedStatement

```java
/**
 *  存放解析映射配置文件的内容
 *     <select id="selectOne" resultType="com.itheima.pojo.User" parameterType="com.itheima.pojo.User">
 *         select * from user where id = #{id} and username = #{username}
 *     </select>
 */
public class MappedStatement {

    // 1.唯一标识 （statementId namespace.id）
    private String statementId;
    // 2.返回结果类型
    private String resultType;
    // 3.参数类型
    private String parameterType;
    // 4.要执行的sql语句
    private String sql;

    // 5.mapper代理：sqlcommandType
    private String sqlcommandType;

    public String getSqlcommandType() {
        return sqlcommandType;
    }

    public void setSqlcommandType(String sqlcommandType) {
        this.sqlcommandType = sqlcommandType;
    }

    public String getStatementId() {
        return statementId;
    }

    public void setStatementId(String statementId) {
        this.statementId = statementId;
    }

    public String getResultType() {
        return resultType;
    }

    public void setResultType(String resultType) {
        this.resultType = resultType;
    }

    public String getParameterType() {
        return parameterType;
    }

    public void setParameterType(String parameterType) {
        this.parameterType = parameterType;
    }

    public String getSql() {
        return sql;
    }

    public void setSql(String sql) {
        this.sql = sql;
    }
}
```



SqlSessionFactoryBuilder

```java
public class SqlSessionFactoryBuilder {

    /**
     * 1.解析配置文件，封装Configuration 2.创建SqlSessionFactory工厂对象
     * @return
     */
    public SqlSessionFactory build(InputStream inputStream) throws DocumentException {
        // 1.解析配置文件，封装Configuration
        XMLConfigBuilder xmlConfigBuilder = new XMLConfigBuilder();
        Configuration configuration = xmlConfigBuilder.parse(inputStream);

        SqlSessionFactory defatultSqlSessionFactory = new DefatultSqlSessionFactory(configuration);
        return  defatultSqlSessionFactory;
    }

}
```

XMLConfigerBuilder

```java
public class XMLConfigBuilder {
    
    private Configuration configuration;

    public XMLConfigBuilder() {
        configuration = new Configuration();
    }

    /**
     * 使用dom4j解析xml文件，封装configuration对象
     * @param inputStream
     * @return
     */
    public Configuration parse(InputStream inputStream) throws DocumentException {

        Document document = new SAXReader().read(inputStream);
        Element rootElement = document.getRootElement();

        // 解析核心配置文件中数据源部分
        List<Element> list = rootElement.selectNodes("//property");
        //  <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>

        Properties properties = new Properties();
        for (Element element : list) {
            String name = element.attributeValue("name");
            String value = element.attributeValue("value");
            properties.setProperty(name,value);
        }

        // 创建数据源对象（连接池）
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setDriverClassName(properties.getProperty("driverClassName"));
        druidDataSource.setUrl(properties.getProperty("url"));
        druidDataSource.setUsername(properties.getProperty("username"));
        druidDataSource.setPassword(properties.getProperty("password"));

        // 创建好的数据源对象封装进configuration中、
        configuration.setDataSource(druidDataSource);


        // 解析映射配置文件
        // 1.获取映射配置文件的路径  2.解析  3.封装好mappedStatement
        List<Element> mapperList = rootElement.selectNodes("//mapper");
        for (Element element : mapperList) {
            String mapperPath = element.attributeValue("resource");
            InputStream resourceAsSteam = Resources.getResourceAsSteam(mapperPath);
            XMLMapperBuilder xmlMapperBuilder = new XMLMapperBuilder(configuration);
            xmlMapperBuilder.parse(resourceAsSteam);
        }

        return configuration;
    }
}
```

XMLMapperBuilder

```java
public class XMLMapperBuilder {
 private Configuration configuration;

    public XMLMapperBuilder(Configuration configuration) {
        this.configuration = configuration;
    }

    public void parse(InputStream inputStream) throws DocumentException, ClassNotFoundException {
        Document document = new SAXReader().read(inputStream);
        Element rootElement = document.getRootElement();

        String namespace = rootElement.attributeValue("namespace");
        List<Element> select = rootElement.selectNodes("select");
        for (Element element : select) { //id的值
            String id = element.attributeValue("id");
            String paramterType = element.attributeValue("paramterType");
            String resultType = element.attributeValue("resultType"); //输入参数class
            Class<?> paramterTypeClass = getClassType(paramterType);
            //返回结果class
            Class<?> resultTypeClass = getClassType(resultType);
            //statementId
            String key = namespace + "." + id;
            //sql语句
            String textTrim = element.getTextTrim();
            //封装 mappedStatement
            MappedStatement mappedStatement = new MappedStatement();
            mappedStatement.setId(id);
            mappedStatement.setParamterType(paramterTypeClass);
            mappedStatement.setResultType(resultTypeClass);
            mappedStatement.setSql(textTrim);
            //填充 configuration
            configuration.getMappedStatementMap().put(key, mappedStatement); 

            private Class<?> getClassType (String paramterType) throws ClassNotFoundException {
                Class<?> aClass = Class.forName(paramterType);
                return aClass;
    }
}
```

sqlSessionFactory 接口及D efaultSqlSessionFactory 实现类

```java
public interface SqlSessionFactory {

    /**
     * 生产sqlSession ：封装着与数据库交互的方法
     * @return
     */
    public SqlSession openSession();

}

public class DefatultSqlSessionFactory implements SqlSessionFactory {

    private Configuration configuration;

    public DefatultSqlSessionFactory(Configuration configuration) {
        this.configuration = configuration;
    }

    @Override
    public SqlSession openSession() {

        // 执行器创建出来
        Executor executor = new SimpleExecutor();

        DefaultSqlSession defaultSqlSession = new DefaultSqlSession(configuration,executor);
        return defaultSqlSession;
    }
}

```

sqlSession 接口及 DefaultSqlSession 实现类

```java
public interface SqlSession {

    /**
     * 查询所有的方法 select * from user where username like '%aaa%' and sex = ''
     * 参数1：唯一标识
     * 参数2：入参
     */
    public <E> List<E> selectList(String statementId,Object parameter) throws Exception;


    /**
     * 查询单个的方法
     */
    public <T> T selectOne(String statementId,Object parameter) throws Exception;
}

public class DefaultSqlSession implements SqlSession {

    private Configuration configuration;

    private Executor executor;

    public DefaultSqlSession(Configuration configuration, Executor executor) {
        this.configuration = configuration;
        this.executor = executor;
    }

    @Override                    // user.selectList      1 tom user
    public <E> List<E> selectList(String statementId, Object params) throws Exception {

        MappedStatement mappedStatement = configuration.getMappedStatementMap().get(statementId);
        // 将查询操作委派给底层的执行器
        List<E> list = executor.query(configuration,mappedStatement,params);

        return list;
    }

    @Override
    public <T> T selectOne(String statementId, Object params) throws Exception {
        List<Object> list = this.selectList(statementId, params);
        if(list.size() == 1){
            return (T) list.get(0);
        }else if(list.size() > 1){
            throw new RuntimeException("返回结果过多");
        }else {
            return null;
        }
    }
}   
```

Executor

```java
public interface Executor {

    <E> List<E> query(Configuration configuration, MappedStatement mappedStatement, Object params) throws Exception;
}

```

SimpleExecutor

```java
public class SimpleExecutor implements Executor {

    /**
     * 执行JDBC操作
     * @param configuration
     * @param mappedStatement
     * @param params
     * @param <E>
     * @return
     */
    @Override                                                                               // user product
    public <E> List<E> query(Configuration configuration, MappedStatement mappedStatement, Object params) throws Exception {

        // 1. 加载驱动，获取连接
        Connection connection = configuration.getDataSource().getConnection();

        // 2. 获取prepareStatement预编译对象
        /*
             select * from user where id = #{id} and username = #{username}
             select * from user where id = ? and username = ?
             占位符替换 ：#{}替换成？ 注意：#{id}里面的id名称要保存
         */
        String sql = mappedStatement.getSql();
        BoundSql boundSql = getBoundSQL(sql);
        String finaLSql = boundSql.getFinaLSql();
        PreparedStatement preparedStatement = connection.prepareStatement(finaLSql);

        // 3.设置参数

        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
        if(parameterMappings.size() > 0){
        // com.itheima.pojo.User
        String parameterType = mappedStatement.getParameterType();
        Class<?> parameterTypeClass = Class.forName(parameterType);

        for (int i = 0; i < parameterMappings.size(); i++) {
            ParameterMapping parameterMapping = parameterMappings.get(i);
            // id
            String content = parameterMapping.getContent();
            // 反射
            Field declaredField = parameterTypeClass.getDeclaredField(content);
            // 暴力访问
            declaredField.setAccessible(true);
            Object value = declaredField.get(params);
            preparedStatement.setObject(i+1 ,value);
        }
        }

        // 4.执行sql,发起查询
        ResultSet resultSet = preparedStatement.executeQuery();
        String resultType = mappedStatement.getResultType();
        Class<?> resultTypeClass = Class.forName(resultType);

        ArrayList<E> list = new ArrayList<>();
        // 5.遍历封装
        while (resultSet.next()){
            // 元数据信息中包含了字段名 字段的值
            ResultSetMetaData metaData = resultSet.getMetaData();
            Object obj = resultTypeClass.newInstance();
            for (int i = 1; i <= metaData.getColumnCount() ; i++) {

                // id  username
                String columnName = metaData.getColumnName(i);
                Object value = resultSet.getObject(columnName);
                // 属性描述器
                PropertyDescriptor propertyDescriptor = new PropertyDescriptor(columnName,resultTypeClass);
                Method writeMethod = propertyDescriptor.getWriteMethod();
                writeMethod.invoke(obj,value);
            }
            list.add((E) obj);
        }
        return list;
    }

    /**
     *  1.将sql中#{}替换成？ 2.将#{}里面的值保存
     * @param sql
     * @return
     */
    private BoundSql getBoundSQL(String sql) {
        // 标记处理器：配合标记解析器完成标记的解析工作
        ParameterMappingTokenHandler parameterMappingTokenHandler = new ParameterMappingTokenHandler();

        // 标记解析器
        GenericTokenParser genericTokenParser = new GenericTokenParser("#{", "}", parameterMappingTokenHandler);
        String finalSql = genericTokenParser.parse(sql);

        // #{}里面的值的集合
        List<ParameterMapping> parameterMappings = parameterMappingTokenHandler.getParameterMappings();

        BoundSql boundSql = new BoundSql(finalSql, parameterMappings);

        return boundSql;
    }
}
```

BoundSql

```java
public class BoundSql {
	 //解析过后的sql语句
    private String sqlText;
    //解析出来的参数
    private List<ParameterMapping> parameterMappingList = new ArrayList<ParameterMapping>();

    public BoundSql(String sqlText, List<ParameterMapping>
            parameterMappingList) {
        this.sqlText = sqlText;
        this.parameterMappingList = parameterMappingList;
    }

    public String getSqlText() {
        return sqlText;
    }

    public void setSqlText(String sqlText) {
        this.sqlText = sqlText;
    }

    public List<ParameterMapping> getParameterMappingList() {
        return parameterMappingList;
    }

    public void setParameterMappingList(List<ParameterMapping> parameterMappingList) {
        this.parameterMappingList = parameterMappingList;
    }
}
```



### **1.5 **自定义持久层框架_优化

通过上述我们的自定义框架，我们解决了JDBC操作数据库带来的一些问题：例如频繁创建释放数据库连 接，硬编码，手动封装返回结果集等问题，但是现在我们继续来分析刚刚完成的自定义框架代码，有没 有什么问题？

问题如下：

* dao的实现类中存在重复的代码，整个操作的过程模板重复(创建sqlsession,调用sqlsession方 法，关闭 sqlsession)

* dao的实现类中存在硬编码，调用sqlsession的方法时，参数statement的id硬编码

解决：使用代理模式来创建接口的代理对象

```java
  @Test
    public void test2() throws Exception {
        InputStream resourceAsSteam = Resources.getResourceAsSteam(path： "sqlMapConfig.xml")
        SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resourceAsSteam);
        SqlSession sqlSession = build.openSession();
        User user = new User();
        user.setld(l);
        user.setUsername("tom");
        //代理对象
        UserMapper userMapper = sqlSession.getMappper(UserMapper.class);
        User userl = userMapper.selectOne(user);
        System・out.println(userl);
    }
```

在sqlSession中添加方法

```java
public interface SqlSession {
   public <T> T getMappper(Class<?> mapperClass);
```

实现类

```java
public class DefaultSqlSession implements SqlSession {

    private Configuration configuration;

    private Executor executor;

    public DefaultSqlSession(Configuration configuration, Executor executor) {
        this.configuration = configuration;
        this.executor = executor;
    }

    @Override
    public <T> T getMapper(Class<?> c) {

        // 基于JDK动态代理产生接口的代理对象
        Object proxy = Proxy.newProxyInstance(DefaultSqlSession.class.getClassLoader(), new Class[]{c}, new InvocationHandler() {

            /*
             o : 代理对象：很少用到
             method ：正在执行的方法
             objects ：方法的参数
             */
            @Override
            public Object invoke(Object o, Method method, Object[] objects) throws Throwable {
                // findByCondition
                String methodName = method.getName();

                // com.itheima.dao.IUserDao
                String className = method.getDeclaringClass().getName();

                // 唯一标识：namespace.id  com.itheima.dao.IUserDao.findByCondition
                String statementId = className + "." +methodName;

                MappedStatement mappedStatement = configuration.getMappedStatementMap().get(statementId);
                String sql = mappedStatement.getSql();

                // sqlcommandType select  insert update  delete
                String sqlcommandType = mappedStatement.getSqlcommandType();
                switch (sqlcommandType){

                    case "select":
                        // 查询操作 问题来了：selectList 还是selectOne?
                        Type genericReturnType = method.getGenericReturnType();
                        // 判断是否实现泛型类型参数化
                        if(genericReturnType instanceof ParameterizedType){
                           return selectList(statementId,objects);
                        }

                             return selectOne(statementId,objects);

                    case "update":
                        break;
                        // 更新操作
                    case "delete":
                        break;
                        // 删除操作
                    case "insert":
                        break;
                        // 添加操作
                }

                return null;
            }
        });


        return (T) proxy;
    }
```





## 2. MyBatis架构原理&主要组件

### 2.1 MyBatis的架构设计

<img src=".\img\image-20210811110200438.png" alt="image-20210811110200438" style="zoom:90%;" />



> mybatis架构四层作用是什么呢？

- Api接口层：提供API 增加、删除、修改、查询等接口，通过API接口对数据库进行操作。
- 数据处理层：主要负责SQL的 查询、解析、执行以及结果映射的处理，主要作用解析sql根据调用请求完成一次数据库操作.
- 框架支撑层：负责通用基础服务支撑,包含事务管理、连接池管理、缓存管理等共用组件的封装，为上层提供基础服务支撑.
- 引导层：引导层是配置和启动MyBatis 配置信息的方式



### 2.2 MyBatis主要组件及其相互关系

组件关系如下图所示：

<img src=".\img\组件.jpg" style="zoom:50%;" />



组件介绍：

* SqlSession：是Mybatis对外暴露的核心API，提供了对数据库的DRUD操作接口。

* Executor：执行器，由SqlSession调用，负责数据库操作以及Mybatis两级缓存的维护

* StatementHandler：封装了JDBC Statement操作，负责对Statement的操作，例如PrepareStatement参数的设置以及结果集的处理。

* ParameterHandler：是StatementHandler内部一个组件，主要负责对ParameterStatement参数的设置

* ResultSetHandler：是StatementHandler内部一个组件，主要负责对ResultSet结果集的处理，封装成目标对象返回

* TypeHandler：用于Java类型与JDBC类型之间的数据转换，ParameterHandler和ResultSetHandler会分别使用到它的类型转换功能

* MappedStatement：是对Mapper配置文件或Mapper接口方法上通过注解申明SQL的封装

* Configuration：Mybatis所有配置都统一由Configuration进行管理，内部由具体对象分别管理各自的小功能模块
  





## 3. 源码剖析-源码环境搭建

### 3.1 源码环境搭建

* mybatis源码地址：https://github.com/mybatis/mybatis-3

![image-20210805095349781](.\img\image-20210805095349781.png)



<img src=".\img\image-20210805104502919.png" alt="image-20210805104502919" style="zoom:80%;" />



<img src=".\img\image-20210805104602906.png" alt="image-20210805104602906" style="zoom:80%;" />





### 3.2 源码导入&编译

<img src=".\img\image-20210805095727859.png" alt="image-20210805095727859" style="zoom:80%;" />



<img src=".\img\源码介绍.png" style="zoom:10%;" />



### 3.3 编写测试代码

<img src=".\img\image-20210805111327843.png" alt="image-20210805111327843" style="zoom:70%;" />



#### 3.3.1 配置sqlMapConfig.xml 

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    
    <!--第一部分：数据源配置-->
    <environments default="development">
        <environment id="development">
            <!-- 使用jdbc事务管理 -->
            <transactionManager type="JDBC" />
            <!-- 数据库连接池 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver" />
                <property name="url"
                          value="jdbc:mysql:///zdy_mybatis?useSSL=false&amp;characterEncoding=UTF-8&amp;serverTimezone=UTC" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>

   <!--第二部分：引入映射配置文件-->
    <mappers>
      <mapper resource="mapper/UserMapper.xml"></mapper>
    </mappers>
    
</configuration>
```



#### 3.3.2 配置UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="user">
 
  <select id="findUserById" parameterType="int" resultType="com.itheima.pojo.User">
        SELECT id,username FROM  user WHERE id = #{id}
    </select>
  
</mapper>
```



#### 3.3.3 编写User类

```java
package com.itheima.pojo;

import lombok.Data;

@Data
public class User {

    // ID标识
    private Integer id;
    // 用户名
    private String username;

}
```



#### 3.3.5 编写测试类

```java
public class MybatisTest {

      @Test
      public void test1() throws IOException {

        InputStream resourceAsStream = Resources.getResourceAsStream("SqlMapConfig.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();

        User user = sqlSession.selectOne("user.findUserById", user1);
        System.out.println(user);
        System.out.println("MyBatis源码环境搭建成功...");
        sqlSession.close();
      }

}
```

输出：

<img src=".\img\image-20210805111558961.png" alt="image-20210805111558961" style="zoom:70%;" />





## 4. 源码剖析-初始化_如何解析的全局配置文件？

### 前言

全局配置文件可配置参数：https://mybatis.org/mybatis-3/zh/configuration.html

<img src=".\img\image-20210805154805841.png" alt="image-20210805154805841" style="zoom:80%;" />



* **Configuration对象**

```java
public class Configuration {

  protected Environment environment;

  // 允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为false。默认为false
  protected boolean safeRowBoundsEnabled;
  // 允许在嵌套语句中使用分页（ResultHandler）。如果允许使用则设置为false
  protected boolean safeResultHandlerEnabled = true;
  // 是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN
  // 到经典 Java 属性名 aColumn 的类似映射。默认false
  protected boolean mapUnderscoreToCamelCase;
  // 当开启时，任何方法的调用都会加载该对象的所有属性。否则，每个属性会按需加载。默认值false (true in ≤3.4.1)
  protected boolean aggressiveLazyLoading;
  // 是否允许单一语句返回多结果集（需要兼容驱动）。
  protected boolean multipleResultSetsEnabled = true;
  // 允许 JDBC 支持自动生成主键，需要驱动兼容。这就是insert时获取mysql自增主键/oracle sequence的开关。
  // 注：一般来说,这是希望的结果,应该默认值为true比较合适。
  protected boolean useGeneratedKeys;
  // 使用列标签代替列名,一般来说,这是希望的结果
  protected boolean useColumnLabel = true;
  // 是否启用缓存
  protected boolean cacheEnabled = true;
  // 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，
  // 这对于有 Map.keySet() 依赖或 null 值初始化的时候是有用的。
  protected boolean callSettersOnNulls;
  // 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的工程必须采用Java 8编译，
  // 并且加上-parameters选项。（从3.4.1开始）
  protected boolean useActualParamName = true;
  //当返回行的所有列都是空时，MyBatis默认返回null。 当开启这个设置时，MyBatis会返回一个空实例。
  // 请注意，它也适用于嵌套的结果集 (i.e. collectioin and association)。（从3.4.2开始）
  // 注：这里应该拆分为两个参数比较合适, 一个用于结果集，一个用于单记录。
  // 通常来说，我们会希望结果集不是null，单记录仍然是null
  protected boolean returnInstanceForEmptyRow;

  protected boolean shrinkWhitespacesInSql;

  // 指定 MyBatis 增加到日志名称的前缀。
  protected String logPrefix;
  // 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。一般建议指定为slf4j或log4j
  protected Class<? extends Log> logImpl;
  // 指定VFS的实现, VFS是mybatis提供的用于访问AS内资源的一个简便接口
  protected Class<? extends VFS> vfsImpl;
  protected Class<?> defaultSqlProviderType;
  // MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。
  // 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。
  // 若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。
  protected LocalCacheScope localCacheScope = LocalCacheScope.SESSION;
  // 当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。 某些驱动需要指定列的 JDBC 类型，
  // 多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。
  protected JdbcType jdbcTypeForNull = JdbcType.OTHER;
  // 指定对象的哪个方法触发一次延迟加载。
  protected Set<String> lazyLoadTriggerMethods = new HashSet<>(Arrays.asList("equals", "clone", "hashCode", "toString"));
  // 设置超时时间，它决定驱动等待数据库响应的秒数。默认不超时
  protected Integer defaultStatementTimeout;
  // 为驱动的结果集设置默认获取数量。
  protected Integer defaultFetchSize;
  // SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）；
  // BATCH 执行器将重用语句并执行批量更新。
  protected ResultSetType defaultResultSetType;

  // 默认执行器类型
  protected ExecutorType defaultExecutorType = ExecutorType.SIMPLE;
  // 指定 MyBatis 应如何自动映射列到字段或属性。
  // NONE 表示取消自动映射；
  // PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。
  // FULL 会自动映射任意复杂的结果集（无论是否嵌套）。
  protected AutoMappingBehavior autoMappingBehavior = AutoMappingBehavior.PARTIAL;
  // 指定发现自动映射目标未知列（或者未知属性类型）的行为。这个值应该设置为WARNING比较合适
  protected AutoMappingUnknownColumnBehavior autoMappingUnknownColumnBehavior = AutoMappingUnknownColumnBehavior.NONE;
  // settings下的properties属性
  protected Properties variables = new Properties();
  // 默认的反射器工厂,用于操作属性、构造器方便
  protected ReflectorFactory reflectorFactory = new DefaultReflectorFactory();
  // 对象工厂, 所有的类resultMap类都需要依赖于对象工厂来实例化
  protected ObjectFactory objectFactory = new DefaultObjectFactory();
  // 对象包装器工厂,主要用来在创建非原生对象,比如增加了某些监控或者特殊属性的代理类
  protected ObjectWrapperFactory objectWrapperFactory = new DefaultObjectWrapperFactory();

  // 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。特定关联关系中可通过设置fetchType属性来覆盖该项的开关状态
  protected boolean lazyLoadingEnabled = false;
  // 指定 Mybatis 创建具有延迟加载能力的对象所用到的代理工具。MyBatis 3.3+使用JAVASSIST
  protected ProxyFactory proxyFactory = new JavassistProxyFactory(); // #224 Using internal Javassist instead of OGNL

  // MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持是基于映射语句中的 databaseId 属性。
  protected String databaseId;
  /**
   * Configuration factory class.
   * Used to create Configuration for loading deserialized unread properties.
   *
   * @see <a href='https://github.com/mybatis/old-google-code-issues/issues/300'>Issue 300 (google code)</a>
   */
  protected Class<?> configurationFactory;

  protected final MapperRegistry mapperRegistry = new MapperRegistry(this);

  // mybatis插件列表
  protected final InterceptorChain interceptorChain = new InterceptorChain();
  protected final TypeHandlerRegistry typeHandlerRegistry = new TypeHandlerRegistry(this);

  // 类型注册器, 用于在执行sql语句的出入参映射以及mybatis-config文件里的各种配置
  // 比如<transactionManager type="JDBC"/><dataSource type="POOLED">时使用简写
  protected final TypeAliasRegistry typeAliasRegistry = new TypeAliasRegistry();
  protected final LanguageDriverRegistry languageRegistry = new LanguageDriverRegistry();

  protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection")
      .conflictMessageProducer((savedValue, targetValue) ->
          ". please check " + savedValue.getResource() + " and " + targetValue.getResource());
  protected final Map<String, Cache> caches = new StrictMap<>("Caches collection");
  protected final Map<String, ResultMap> resultMaps = new StrictMap<>("Result Maps collection");
  protected final Map<String, ParameterMap> parameterMaps = new StrictMap<>("Parameter Maps collection");
  protected final Map<String, KeyGenerator> keyGenerators = new StrictMap<>("Key Generators collection");

  protected final Set<String> loadedResources = new HashSet<>();
  protected final Map<String, XNode> sqlFragments = new StrictMap<>("XML fragments parsed from previous mappers");

  protected final Collection<XMLStatementBuilder> incompleteStatements = new LinkedList<>();
  protected final Collection<CacheRefResolver> incompleteCacheRefs = new LinkedList<>();
  protected final Collection<ResultMapResolver> incompleteResultMaps = new LinkedList<>();
  protected final Collection<MethodResolver> incompleteMethods = new LinkedList<>();

  /*
   * A map holds cache-ref relationship. The key is the namespace that
   * references a cache bound to another namespace and the value is the
   * namespace which the actual cache is bound to.
   */
  protected final Map<String, String> cacheRefMap = new HashMap<>();

  public Configuration(Environment environment) {
    this();
    this.environment = environment;
  }

```



问题：核心配置文件&映射配置文件如何被解析的？

### 解析配置文件源码流程：

### 入口：SqlSessionFactoryBuilder#build

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
      // XMLConfigBuilder:用来解析XML配置文件
      // 使用构建者模式
      XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      // parser.parse()：使用XPATH解析XML配置文件，将配置文件封装为Configuration对象
      // 返回DefaultSqlSessionFactory对象，该对象拥有Configuration对象（封装配置文件信息）
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        inputStream.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }
```



创建XMLConfigBuilder对象，这个类是BaseBuilder的子类，BaseBuilder类图：

![image-20210805160248237](.\img\image-20210805160248237.png)

看到这些子类基本上都是以Builder结尾，这里使用的是**Builder建造者设计模式**。



### XMLConfigBuilder#构造参数

XMLConfigBuilder:用来解析XML配置文件(使用构建者模式)

```java
// XMLConfigBuilder:用来解析XML配置文件
// 使用构建者模式
XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
```

```java
 public XMLConfigBuilder(InputStream inputStream, String environment, Properties props) {
    this(new XPathParser(inputStream, true, props, new XMLMapperEntityResolver()), environment, props);
```

Mybatis对应解析包org.apache.ibatis.parsing:

<img src=".\img\image-20210805160823970.png" alt="image-20210805160823970" style="zoom:80%;" />

XPathParser基于 Java XPath 解析器，用于解析 MyBatis中

- SqlMapConfig.xml
- mapper.xml

XPathParser主要内容：

<img src=".\img\image-20210805160904487.png" alt="image-20210805160904487" style="zoom:80%;" />



#### 1. XpathParser#构造函数

用来使用XPath语法解析XML的解析器

```java
public XPathParser(InputStream inputStream, boolean validation, Properties variables, EntityResolver entityResolver) {
    commonConstructor(validation, variables, entityResolver);
    // 解析XML文档为Document对象
    this.document = createDocument(new InputSource(inputStream));
  }
```



##### 1.1 XPathParser#createDocument

解析全局配置文件，封装为Document对象（封装一些子节点，使用XPath语法解析获取）

```java
private Document createDocument(InputSource inputSource) {
    // important: this must only be called AFTER common constructor
    try {
      DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
      // 进行dtd或者Schema校验
      factory.setValidating(validation);

      factory.setNamespaceAware(false);
      // 设置忽略注释为true
      factory.setIgnoringComments(true);
      // 设置是否忽略元素内容中的空白
      factory.setIgnoringElementContentWhitespace(false);
      factory.setCoalescing(false);
      factory.setExpandEntityReferences(true);

      DocumentBuilder builder = factory.newDocumentBuilder();
      builder.setEntityResolver(entityResolver);
      builder.setErrorHandler(new ErrorHandler() {
        @Override
        public void error(SAXParseException exception) throws SAXException {
          throw exception;
        }

        @Override
        public void fatalError(SAXParseException exception) throws SAXException {
          throw exception;
        }

        @Override
        public void warning(SAXParseException exception) throws SAXException {
        }
      });
      // 通过dom解析，获取Document对象
      return builder.parse(inputSource);
    } catch (Exception e) {
      throw new BuilderException("Error creating document instance.  Cause: " + e, e);
    }
  }
```



#### 2. XMLConfigBuilder#构造函数

创建Configuration对象，同时初始化内置类的别名

```java
private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
    //  创建Configuration对象，并通过TypeAliasRegistry注册一些Mybatis内部相关类的别名
    super(new Configuration());
    ErrorContext.instance().resource("SQL Mapper Configuration");
    this.configuration.setVariables(props);
    this.parsed = false;
    this.environment = environment;
    this.parser = parser;
  }
```



##### 2.1 Configuration#构造函数

创建Configuration对象，同时初始化内置类的别名

```java
public Configuration() {
        //TypeAliasRegistry（类型别名注册器）
        typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
        typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);

        typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);
        typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
        typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);

        typeAliasRegistry.registerAlias("PERPETUAL", PerpetualCache.class);
        typeAliasRegistry.registerAlias("FIFO", FifoCache.class);
        typeAliasRegistry.registerAlias("LRU", LruCache.class);
        typeAliasRegistry.registerAlias("SOFT", SoftCache.class);
        typeAliasRegistry.registerAlias("WEAK", WeakCache.class);

        typeAliasRegistry.registerAlias("DB_VENDOR", VendorDatabaseIdProvider.class);

        typeAliasRegistry.registerAlias("XML", XMLLanguageDriver.class);
        typeAliasRegistry.registerAlias("RAW", RawLanguageDriver.class);

        typeAliasRegistry.registerAlias("SLF4J", Slf4jImpl.class);
        typeAliasRegistry.registerAlias("COMMONS_LOGGING", JakartaCommonsLoggingImpl.class);
        typeAliasRegistry.registerAlias("LOG4J", Log4jImpl.class);
        typeAliasRegistry.registerAlias("LOG4J2", Log4j2Impl.class);
        typeAliasRegistry.registerAlias("JDK_LOGGING", Jdk14LoggingImpl.class);
        typeAliasRegistry.registerAlias("STDOUT_LOGGING", StdOutImpl.class);
        typeAliasRegistry.registerAlias("NO_LOGGING", NoLoggingImpl.class);

        typeAliasRegistry.registerAlias("CGLIB", CglibProxyFactory.class);
        typeAliasRegistry.registerAlias("JAVASSIST", JavassistProxyFactory.class);

        languageRegistry.setDefaultDriverClass(XMLLanguageDriver.class);
        languageRegistry.register(RawLanguageDriver.class);
    }
```

- ### XMLConfigBuilder#parse

```java
//使用XPATH解析XML配置文件，将配置文件封装为Configuration对象
parser.parse();
```



### XMLConfigBuilder#parse

解析XML配置文件

```java
/**
   * 解析XML配置文件
   * @return
   */
  public Configuration parse() {
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    // parser.evalNode("/configuration")：通过XPATH解析器，解析configuration根节点
    // 从configuration根节点开始解析，最终将解析出的内容封装到Configuration对象中
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }
```

#### 1. XPathParser#evalNode（xpath语法）

XPath解析器，专门用来通过Xpath语法解析XML返回XNode节点

```java
public XNode evalNode(String expression) {
    // 根据XPATH语法，获取指定节点
    return evalNode(document, expression);
  }

  public XNode evalNode(Object root, String expression) {
    Node node = (Node) evaluate(expression, root, XPathConstants.NODE);
    if (node == null) {
      return null;
    }
    return new XNode(this, node, variables);
  }
```



#### 2. XMLConfigBuilder#parseConfiguration(XNode)

从configuration根节点开始解析，最终将解析出的内容封装到Configuration对象中

```java
private void parseConfiguration(XNode root) {
    try {
      //issue #117 read properties first
      // 解析</properties>标签
      propertiesElement(root.evalNode("properties"));
      // 解析</settings>标签
      Properties settings = settingsAsProperties(root.evalNode("settings"));
      loadCustomVfs(settings);
      loadCustomLogImpl(settings);
      // 解析</typeAliases>标签
      typeAliasesElement(root.evalNode("typeAliases"));
      // 解析</plugins>标签
      pluginElement(root.evalNode("plugins"));
      // 解析</objectFactory>标签
      objectFactoryElement(root.evalNode("objectFactory"));
      // 解析</objectWrapperFactory>标签
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      // 解析</reflectorFactory>标签
      reflectorFactoryElement(root.evalNode("reflectorFactory"));
      settingsElement(settings);
      
      // read it after objectFactory and objectWrapperFactory issue #631
      // 解析</environments>标签
      environmentsElement(root.evalNode("environments"));
      // 解析</databaseIdProvider>标签
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      // 解析</typeHandlers>标签
      typeHandlerElement(root.evalNode("typeHandlers"));
      // 解析</mappers>标签 加载映射文件流程主入口
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```

- ### SqlSessionFactoryBuilder#build

返回DefaultSqlSessionFactory对象，该对象拥有Configuration对象（封装配置文件信息）

```java
// 返回DefaultSqlSessionFactory对象，该对象拥有Configuration对象（封装配置文件信息）
return build(parser.parse());
```

```java
public SqlSessionFactory build(Configuration config) {
    // 创建SqlSessionFactory接口的默认实现类
    return new DefaultSqlSessionFactory(config);
  }
```



**总结**

<img src=".\img\image-20210805161938370.png" alt="image-20210805161938370" style="zoom:80%;" />





## 5. 源码剖析-初始化_如何解析的映射配置文件？

### 前言

![image-20210805155009169](.\img\image-20210805155009169.png)

- ### select

select 元素允许你配置很多属性来配置每条语句的行为细节

```xml
<select
  id="select"
  parameterType="int"
  parameterMap="deprecated"
  resultType="hashmap"
  resultMap="personResultMap"
  flushCache="false"
  useCache="true"
  timeout="10"
  fetchSize="256"
  statementType="PREPARED"
  resultSetType="FORWARD_ONLY">
```



* ### insert, update 和 delete

数据变更语句 insert，update 和 delete 的实现非常接近

```xml
<insert
  id="insert"
  parameterType="com.itheima.pojo.User"
  flushCache="true"
  statementType="PREPARED"
  keyProperty=""
  keyColumn=""
  useGeneratedKeys=""
  timeout="20">

<update
  id="update"
  parameterType="com.itheima.pojo.User"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">

<delete
  id="delete"
  parameterType="com.itheima.pojo.User"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">
```



- ### 动态sql

借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类

- if
- choose (when, otherwise)
   MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```



问题：映射配置文件中标签和属性如何被解析封装的？

问题：sql占位符如何进行的替换？动态sql如何进行的解析？

### 解析映射配置文件源码流程：

### 入口：XMLConfigBuilder#mapperElement

解析全局配置文件中的\<mappers>标签

```java
/**
   * 解析<mappers>标签
   * @param parent  mappers标签对应的XNode对象
   * @throws Exception
   */
  private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      // 获取<mappers>标签的子标签
      for (XNode child : parent.getChildren()) {
        // <package>子标签
        if ("package".equals(child.getName())) {
          // 获取mapper接口和mapper映射文件对应的package包名
          String mapperPackage = child.getStringAttribute("name");
          // 将包下所有的mapper接口以及它的代理对象存储到一个Map集合中，key为mapper接口类型，value为代理对象工厂
          configuration.addMappers(mapperPackage);
        } else {// <mapper>子标签
          // 获取<mapper>子标签的resource属性
          String resource = child.getStringAttribute("resource");
          // 获取<mapper>子标签的url属性
          String url = child.getStringAttribute("url");
          // 获取<mapper>子标签的class属性
          String mapperClass = child.getStringAttribute("class");
          // 它们是互斥的
          if (resource != null && url == null && mapperClass == null) {
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            // 专门用来解析mapper映射文件
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            // 通过XMLMapperBuilder解析mapper映射文件
            mapperParser.parse();
          } else if (resource == null && url != null && mapperClass == null) {
            ErrorContext.instance().resource(url);
            InputStream inputStream = Resources.getUrlAsStream(url);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
            // 通过XMLMapperBuilder解析mapper映射文件
            mapperParser.parse();
          } else if (resource == null && url == null && mapperClass != null) {
            Class<?> mapperInterface = Resources.classForName(mapperClass);
            // 将指定mapper接口以及它的代理对象存储到一个Map集合中，key为mapper接口类型，value为代理对象工厂
            configuration.addMapper(mapperInterface);
          } else {
            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
          }
        }
      }
    }
  }
```



### <package\>子标签

#### 1. Configuration#addMappers

将包下所有的mapper接口以及它的代理对象存储到一个Map集合中，key为mapper接口类型，value为代理对象工厂

```java
public void addMappers(String packageName) {
    mapperRegistry.addMappers(packageName);
}
```

#### 1.1 MapperRegistry#addMappers

将Mapper接口添加到MapperRegistry中

```java
//1
public void addMappers(String packageName) {
    addMappers(packageName, Object.class);
}

//2
public void addMappers(String packageName, Class<?> superType) {
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<>();
    // 根据package名称，加载该包下Mapper接口文件（不是映射文件）
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    // 获取加载的Mapper接口
    Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();
    for (Class<?> mapperClass : mapperSet) {
      // 将Mapper接口添加到MapperRegistry中
      addMapper(mapperClass);
    }
  }

//3
public <T> void addMapper(Class<T> type) {
    if (type.isInterface()) {
      // 如果Map集合中已经有该mapper接口的映射，就不需要再存储了
      if (hasMapper(type)) {
        throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
      }
      boolean loadCompleted = false;
      try {
        // 将mapper接口以及它的代理对象存储到一个Map集合中，key为mapper接口类型，value为代理对象工厂
        knownMappers.put(type, new MapperProxyFactory<T>(type));
        // It's important that the type is added before the parser is run
        // otherwise the binding may automatically be attempted by the
        // mapper parser. If the type is already known, it won't try.
        
        // 用来解析注解方式的mapper接口
        MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
        // 解析注解方式的mapper接口
        parser.parse();
        loadCompleted = true;
      } finally {
        if (!loadCompleted) {
          knownMappers.remove(type);
        }
      }
    }
  }
```



#### 1.1.1 MapperAnnotationBuilder#parse

解析注解方式的mapper接口

```java
public void parse() {
    // 获取mapper接口的全路径
    String resource = type.toString();
    // 是否解析过该mapper接口
    if (!configuration.isResourceLoaded(resource)) {
      // 先解析mapper映射文件
      loadXmlResource();
      // 设置解析标识
      configuration.addLoadedResource(resource);
      // Mapper构建者助手
      assistant.setCurrentNamespace(type.getName());
      // 解析CacheNamespace注解
      parseCache();
      // 解析CacheNamespaceRef注解
      parseCacheRef();
      Method[] methods = type.getMethods();
      for (Method method : methods) {
        try {
          // issue #237
          if (!method.isBridge()) {
            // 每个mapper接口中的方法，都解析成MappedStatement对象
            parseStatement(method);
          }
        } catch (IncompleteElementException e) {
          configuration.addIncompleteMethod(new MethodResolver(this, method));
        }
      }
    }
    //去检查所有的incompleteMethods,如果可以解析了.那就移除
    parsePendingMethods();
  }
```



#### 1.1.1.1 MapperAnnotationBuilder#parseStatement

每个mapper接口中的方法，都解析成MappedStatement对象

```java
void parseStatement(Method method) {
    // 获取Mapper接口的形参类型
    Class<?> parameterTypeClass = getParameterType(method);
    // 解析Lang注解
    LanguageDriver languageDriver = getLanguageDriver(method);
    // 
    SqlSource sqlSource = getSqlSourceFromAnnotations(method, parameterTypeClass, languageDriver);
    if (sqlSource != null) {
      Options options = method.getAnnotation(Options.class);
      // 组装mappedStatementId
      final String mappedStatementId = type.getName() + "." + method.getName();
      Integer fetchSize = null;
      Integer timeout = null;
      StatementType statementType = StatementType.PREPARED;
      ResultSetType resultSetType = null;
      // 获取该mapper接口中的方法是CRUD操作的哪一种
      SqlCommandType sqlCommandType = getSqlCommandType(method);
      // 是否是SELECT操作
      boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
      boolean flushCache = !isSelect;
      boolean useCache = isSelect;

      // 主键生成器，用于主键返回
      KeyGenerator keyGenerator;
      String keyProperty = null;
      String keyColumn = null;
      if (SqlCommandType.INSERT.equals(sqlCommandType) || SqlCommandType.UPDATE.equals(sqlCommandType)) {
        // first check for SelectKey annotation - that overrides everything else
        SelectKey selectKey = method.getAnnotation(SelectKey.class);
        if (selectKey != null) {
          keyGenerator = handleSelectKeyAnnotation(selectKey, mappedStatementId, getParameterType(method), languageDriver);
          keyProperty = selectKey.keyProperty();
        } else if (options == null) {
          keyGenerator = configuration.isUseGeneratedKeys() ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
        } else {
          keyGenerator = options.useGeneratedKeys() ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
          keyProperty = options.keyProperty();
          keyColumn = options.keyColumn();
        }
      } else {
        keyGenerator = NoKeyGenerator.INSTANCE;
      }

      if (options != null) {
        if (FlushCachePolicy.TRUE.equals(options.flushCache())) {
          flushCache = true;
        } else if (FlushCachePolicy.FALSE.equals(options.flushCache())) {
          flushCache = false;
        }
        useCache = options.useCache();
        fetchSize = options.fetchSize() > -1 || options.fetchSize() == Integer.MIN_VALUE ? options.fetchSize() : null; //issue #348
        timeout = options.timeout() > -1 ? options.timeout() : null;
        statementType = options.statementType();
        resultSetType = options.resultSetType();
      }

      // 处理ResultMap注解
      String resultMapId = null;
      ResultMap resultMapAnnotation = method.getAnnotation(ResultMap.class);
      if (resultMapAnnotation != null) {
        String[] resultMaps = resultMapAnnotation.value();
        StringBuilder sb = new StringBuilder();
        for (String resultMap : resultMaps) {
          if (sb.length() > 0) {
            sb.append(",");
          }
          sb.append(resultMap);
        }
        resultMapId = sb.toString();
      } else if (isSelect) {
        resultMapId = parseResultMap(method);
      }

      // 通过Mapper构建助手，创建一个MappedStatement对象，封装信息
      assistant.addMappedStatement(
          mappedStatementId,
          sqlSource,
          statementType,
          sqlCommandType,
          fetchSize,
          timeout,
          // ParameterMapID
          null,
          parameterTypeClass,
          resultMapId,
          getReturnType(method),
          resultSetType,
          flushCache,
          useCache,
          // TODO gcode issue #577
          false,
          keyGenerator,
          keyProperty,
          keyColumn,
          // DatabaseID
          null,
          languageDriver,
          // ResultSets
          options != null ? nullOrEmpty(options.resultSets()) : null);
    }
  }
```



#### 1.1.1.1.2 MapperBuilderAssistant#addMappedStatement

通过Mapper构建助手，创建一个MappedStatement对象，封装信息

```java
public MappedStatement addMappedStatement(
      String id,
      SqlSource sqlSource,
      StatementType statementType,
      SqlCommandType sqlCommandType,
      Integer fetchSize,
      Integer timeout,
      String parameterMap,
      Class<?> parameterType,
      String resultMap,
      Class<?> resultType,
      ResultSetType resultSetType,
      boolean flushCache,
      boolean useCache,
      boolean resultOrdered,
      KeyGenerator keyGenerator,
      String keyProperty,
      String keyColumn,
      String databaseId,
      LanguageDriver lang,
      String resultSets) {

    if (unresolvedCacheRef) {
      throw new IncompleteElementException("Cache-ref not yet resolved");
    }

    id = applyCurrentNamespace(id, false);
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;

    //利用构建者模式，去创建MappedStatement.Builder,用于创建MappedStatement对象
    MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration, id, sqlSource, sqlCommandType)
        .resource(resource)
        .fetchSize(fetchSize)
        .timeout(timeout)
        .statementType(statementType)
        .keyGenerator(keyGenerator)
        .keyProperty(keyProperty)
        .keyColumn(keyColumn)
        .databaseId(databaseId)
        .lang(lang)
        .resultOrdered(resultOrdered)
        .resultSets(resultSets)
        .resultMaps(getStatementResultMaps(resultMap, resultType, id))
        .resultSetType(resultSetType)
        .flushCacheRequired(valueOrDefault(flushCache, !isSelect))
        .useCache(valueOrDefault(useCache, isSelect))
        .cache(currentCache);

    ParameterMap statementParameterMap = getStatementParameterMap(parameterMap, parameterType, id);
    if (statementParameterMap != null) {
      statementBuilder.parameterMap(statementParameterMap);
    }

    // 通过MappedStatement.Builder，构建一个MappedStatement
    MappedStatement statement = statementBuilder.build();
    // 将MappedStatement对象存储到Configuration中的Map集合中，key为statement的id，value为MappedStatement对象
    configuration.addMappedStatement(statement);
    return statement;
  }
```



### <mapper\>子标签

#### 1.XMLMapperBuilder#构造函数

专门用来解析mapper映射文件

```java
public XMLMapperBuilder(InputStream inputStream, Configuration configuration, String resource, Map<String, XNode> sqlFragments) {
    this(new XPathParser(inputStream, true, configuration.getVariables(), new XMLMapperEntityResolver()),
        configuration, resource, sqlFragments);
  }
```



#### 1.1 XPathParser#构造函数

用来使用XPath语法解析XML的解析器

```java
public XPathParser(InputStream inputStream, boolean validation, Properties variables, EntityResolver entityResolver) {
    commonConstructor(validation, variables, entityResolver);
    // 解析XML文档为Document对象
    this.document = createDocument(new InputSource(inputStream));
  }
```



#### 1.1.1 XPathParser#createDocument

创建Mapper映射文件对应的Document对象

```java
private Document createDocument(InputSource inputSource) {
    // important: this must only be called AFTER common constructor
    try {
      DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
      // 进行dtd或者Schema校验
      factory.setValidating(validation);

      factory.setNamespaceAware(false);
      // 设置忽略注释为true
      factory.setIgnoringComments(true);
      // 设置是否忽略元素内容中的空白
      factory.setIgnoringElementContentWhitespace(false);
      factory.setCoalescing(false);
      factory.setExpandEntityReferences(true);

      DocumentBuilder builder = factory.newDocumentBuilder();
      builder.setEntityResolver(entityResolver);
      builder.setErrorHandler(new ErrorHandler() {
        @Override
        public void error(SAXParseException exception) throws SAXException {
          throw exception;
        }

        @Override
        public void fatalError(SAXParseException exception) throws SAXException {
          throw exception;
        }

        @Override
        public void warning(SAXParseException exception) throws SAXException {
        }
      });
      // 通过dom解析，获取Document对象
      return builder.parse(inputSource);
    } catch (Exception e) {
      throw new BuilderException("Error creating document instance.  Cause: " + e, e);
    }
  }
```



#### 1.2 XMLMapperBuilder#构造函数

```java
private XMLMapperBuilder(XPathParser parser, Configuration configuration, String resource, Map<String, XNode> sqlFragments) {
    super(configuration);
    this.builderAssistant = new MapperBuilderAssistant(configuration, resource);
    this.parser = parser;
    this.sqlFragments = sqlFragments;
    this.resource = resource;
  }
```



#### 1.2.1MapperBuilderAssistant#构造函数

用于构建MappedStatement对象的

```java
public MapperBuilderAssistant(Configuration configuration, String resource) {
    super(configuration);
    ErrorContext.instance().resource(resource);
    this.resource = resource;
  }
```



#### 2. XMLMapperBuilder#parse

通过XMLMapperBuilder解析mapper映射文件

```java
public void parse() {
   // mapper映射文件是否已经加载过
   if (!configuration.isResourceLoaded(resource)) {
     // 从映射文件中的\<mapper>根标签开始解析，直到完整的解析完毕
     configurationElement(parser.evalNode("/mapper"));
     // 标记已经解析
     configuration.addLoadedResource(resource);
     bindMapperForNamespace();
   }

   parsePendingResultMaps();
   parsePendingCacheRefs();
   parsePendingStatements();
 }
```



#### 2.1 XMLMapperBuilder#configurationElement

从映射文件中的\<mapper>根标签开始解析，直到完整的解析完毕

```java
 /**
   * 解析映射文件
   * @param context 映射文件根节点<mapper>对应的XNode
   */
  private void configurationElement(XNode context) {
    try {
      // 获取<mapper>标签的namespace值，也就是命名空间
      String namespace = context.getStringAttribute("namespace");
      // 命名空间不能为空
      if (namespace == null || namespace.equals("")) {
        throw new BuilderException("Mapper's namespace cannot be empty");
      }
      
      // 设置当前的命名空间为namespace的值
      builderAssistant.setCurrentNamespace(namespace);
      // 解析<cache-ref>子标签
      cacheRefElement(context.evalNode("cache-ref"));
      // 解析<cache>子标签
      cacheElement(context.evalNode("cache"));
      
      // 解析<parameterMap>子标签
      parameterMapElement(context.evalNodes("/mapper/parameterMap"));
      // 解析<resultMap>子标签
      resultMapElements(context.evalNodes("/mapper/resultMap"));
      // 解析<sql>子标签，也就是SQL片段
      sqlElement(context.evalNodes("/mapper/sql"));
      // 解析<select>\<insert>\<update>\<delete>子标签
      buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
    }
  }
```



#### 2.1.1 XMLMapperBuilder#buildStatementFromContext

用来创建MappedStatement对象的

```java
//1、构建MappedStatement
private void buildStatementFromContext(List<XNode> list) {
    if (configuration.getDatabaseId() != null) {
      buildStatementFromContext(list, configuration.getDatabaseId());
    }
    // 构建MappedStatement
    buildStatementFromContext(list, null);
  }

//2、专门用来解析MappedStatement
private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
    for (XNode context : list) {
      // MappedStatement解析器
      final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
      try {
        // 解析select等4个标签，创建MappedStatement对象
        statementParser.parseStatementNode();
      } catch (IncompleteElementException e) {
        configuration.addIncompleteStatement(statementParser);
      }
    }
  }
```



#### 2.1.1.1 XMLStatementBuilder#构造函数

专门用来解析MappedStatement

```java
public XMLStatementBuilder(Configuration configuration, MapperBuilderAssistant builderAssistant, XNode context, String databaseId) {
    super(configuration);
    this.builderAssistant = builderAssistant;
    this.context = context;
    this.requiredDatabaseId = databaseId;
  }
```



#### 2.1.1.2 XMLStatementBuilder#parseStatementNode

解析<select><insert><update><delete>子标签

```java
/**
   * 解析<select>\<insert>\<update>\<delete>子标签
   */
  public void parseStatementNode() {
    // 获取statement的id属性（特别关键的值）
    String id = context.getStringAttribute("id");
    String databaseId = context.getStringAttribute("databaseId");

    if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {
      return;
    }

    Integer fetchSize = context.getIntAttribute("fetchSize");
    Integer timeout = context.getIntAttribute("timeout");
    String parameterMap = context.getStringAttribute("parameterMap");
    // 获取入参类型
    String parameterType = context.getStringAttribute("parameterType");
    // 别名处理，获取入参对应的Java类型
    Class<?> parameterTypeClass = resolveClass(parameterType);
    // 获取ResultMap
    String resultMap = context.getStringAttribute("resultMap");
    // 获取结果映射类型
    String resultType = context.getStringAttribute("resultType");
    String lang = context.getStringAttribute("lang");
    LanguageDriver langDriver = getLanguageDriver(lang);
    
    // 别名处理，获取返回值对应的Java类型
    Class<?> resultTypeClass = resolveClass(resultType);
    String resultSetType = context.getStringAttribute("resultSetType");
    
    // 设置默认StatementType为Prepared，该参数指定了后面的JDBC处理时，采用哪种Statement
    StatementType statementType = StatementType.valueOf(context.getStringAttribute("statementType", StatementType.PREPARED.toString()));
    ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);

    String nodeName = context.getNode().getNodeName();
    // 解析SQL命令类型是什么？确定操作是CRUD中的哪一种
    SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
    //是否查询语句
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
    boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);
    boolean useCache = context.getBooleanAttribute("useCache", isSelect);
    boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);

    // Include Fragments before parsing
    // <include>标签解析
    XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);
    includeParser.applyIncludes(context.getNode());

    // Parse selectKey after includes and remove them.
    // 解析<selectKey>标签
    processSelectKeyNodes(id, parameterTypeClass, langDriver);
    
    // Parse the SQL (pre: <selectKey> and <include> were parsed and removed)
    // 创建SqlSource，解析SQL，封装SQL语句（未参数绑定）和入参信息
    SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);
   
    String resultSets = context.getStringAttribute("resultSets");
    String keyProperty = context.getStringAttribute("keyProperty");
    String keyColumn = context.getStringAttribute("keyColumn");
    KeyGenerator keyGenerator;
    String keyStatementId = id + SelectKeyGenerator.SELECT_KEY_SUFFIX;
    keyStatementId = builderAssistant.applyCurrentNamespace(keyStatementId, true);
    if (configuration.hasKeyGenerator(keyStatementId)) {
      keyGenerator = configuration.getKeyGenerator(keyStatementId);
    } else {
      keyGenerator = context.getBooleanAttribute("useGeneratedKeys",
          configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType))
          ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
    }

    // 通过构建者助手，创建MappedStatement对象
    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
        resultSetTypeEnum, flushCache, useCache, resultOrdered, 
        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
  }
```



#### 2.1.1.2.1 MapperBuilderAssistant#addMappedStatement

通过构建者助手，创建MappedStatement对象

```java
public MappedStatement addMappedStatement(
      String id,
      SqlSource sqlSource,
      StatementType statementType,
      SqlCommandType sqlCommandType,
      Integer fetchSize,
      Integer timeout,
      String parameterMap,
      Class<?> parameterType,
      String resultMap,
      Class<?> resultType,
      ResultSetType resultSetType,
      boolean flushCache,
      boolean useCache,
      boolean resultOrdered,
      KeyGenerator keyGenerator,
      String keyProperty,
      String keyColumn,
      String databaseId,
      LanguageDriver lang,
      String resultSets) {

    if (unresolvedCacheRef) {
      throw new IncompleteElementException("Cache-ref not yet resolved");
    }

    id = applyCurrentNamespace(id, false);
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;

    //利用构建者模式，去创建MappedStatement.Builder,用于创建MappedStatement对象
    MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration, id, sqlSource, sqlCommandType)
        .resource(resource)
        .fetchSize(fetchSize)
        .timeout(timeout)
        .statementType(statementType)
        .keyGenerator(keyGenerator)
        .keyProperty(keyProperty)
        .keyColumn(keyColumn)
        .databaseId(databaseId)
        .lang(lang)
        .resultOrdered(resultOrdered)
        .resultSets(resultSets)
        .resultMaps(getStatementResultMaps(resultMap, resultType, id))
        .resultSetType(resultSetType)
        .flushCacheRequired(valueOrDefault(flushCache, !isSelect))
        .useCache(valueOrDefault(useCache, isSelect))
        .cache(currentCache);

    ParameterMap statementParameterMap = getStatementParameterMap(parameterMap, parameterType, id);
    if (statementParameterMap != null) {
      statementBuilder.parameterMap(statementParameterMap);
    }

    // 通过MappedStatement.Builder，构建一个MappedStatement
    MappedStatement statement = statementBuilder.build();
    // 将MappedStatement对象存储到Configuration中的Map集合中，key为statement的id，value为MappedStatement对象
    configuration.addMappedStatement(statement);
    return statement;
  }
```



#### 2.1.1.2.1.1 MappedStatement.Builder#构造函数

利用构建者模式，去创建MappedStatement.Builder,用于创建MappedStatement对象

```
public Builder(Configuration configuration, String id, SqlSource sqlSource, SqlCommandType sqlCommandType) {
      mappedStatement.configuration = configuration;
      mappedStatement.id = id;
      mappedStatement.sqlSource = sqlSource;
      mappedStatement.statementType = StatementType.PREPARED;
      mappedStatement.resultSetType = ResultSetType.DEFAULT;
      mappedStatement.parameterMap = new ParameterMap.Builder(configuration, "defaultParameterMap", null, new ArrayList<>()).build();
      mappedStatement.resultMaps = new ArrayList<>();
      mappedStatement.sqlCommandType = sqlCommandType;
      mappedStatement.keyGenerator = configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType) ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
      String logId = id;
      if (configuration.getLogPrefix() != null) {
        logId = configuration.getLogPrefix() + id;
      }
      mappedStatement.statementLog = LogFactory.getLog(logId);
      mappedStatement.lang = configuration.getDefaultScriptingLanguageInstance();
    }
```



#### 2.1.1.2.1.2 MappedStatement#build

通过MappedStatement.Builder，构建一个MappedStatement

```java
public MappedStatement build() {
      assert mappedStatement.configuration != null;
      assert mappedStatement.id != null;
      assert mappedStatement.sqlSource != null;
      assert mappedStatement.lang != null;
      mappedStatement.resultMaps = Collections.unmodifiableList(mappedStatement.resultMaps);
      return mappedStatement;
    }
```



## 6. 源码剖析-SqlSource创建流程

问题：sql占位符如何进行的替换？动态sql如何进行的解析？

### 相关类及对象

* XMLLanguageDriver
* XMLScriptBuilder
* SqlSource接口
* SqlSourceBuilder
* DynamicSqlSource：主要是封装动态SQL标签解析之后的SQL语句和带有${}的SQL语句
* RawSqlSource：主要封装带有#{}的SQL语句
* StaticSqlSource：是BoundSql中要存储SQL语句的一个载体，上面两个SqlSource的SQL语句，最终都会存储到该SqlSource实现类



<img src=".\img\image-20210826172633982.png" alt="image-20210826172633982" style="zoom:67%;" />





```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = #{ACTIVE}
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```



### SqlSource创建流程

#### 入口：XMLLanguageDriver#createSqlSource

创建SqlSource，解析SQL，封装SQL语句（未参数绑定）和入参信息

```java
@Override
    public SqlSource createSqlSource(Configuration configuration, XNode script, Class<?> parameterType) {
        // 初始化了动态SQL标签处理器
        XMLScriptBuilder builder = new XMLScriptBuilder(configuration, script, parameterType);
        // 解析动态SQL
        return builder.parseScriptNode();
    }
```



#### XMLScriptBuilder#构造函数

初始化了动态SQL标签处理器

```java
public XMLScriptBuilder(Configuration configuration, XNode context, Class<?> parameterType) {
        super(configuration);
        this.context = context;
        this.parameterType = parameterType;
        // 初始化动态SQL中的节点处理器集合
        initNodeHandlerMap();
    }
```

#### 1.XMLScriptBuilder#initNodeHandlerMap

初始化动态SQL中的节点处理器集合

```java
private void initNodeHandlerMap() {
        nodeHandlerMap.put("trim", new TrimHandler());
        nodeHandlerMap.put("where", new WhereHandler());
        nodeHandlerMap.put("set", new SetHandler());
        nodeHandlerMap.put("foreach", new ForEachHandler());
        nodeHandlerMap.put("if", new IfHandler());
        nodeHandlerMap.put("choose", new ChooseHandler());
        nodeHandlerMap.put("when", new IfHandler());
        nodeHandlerMap.put("otherwise", new OtherwiseHandler());
        nodeHandlerMap.put("bind", new BindHandler());
    }
```



#### XMLScriptBuilder#parseScriptNode

解析动态SQL

```java
public SqlSource parseScriptNode() {
        // 解析select\insert\ update\delete标签中的SQL语句，最终将解析到的SqlNode封装到MixedSqlNode中的List集合中
        // ****将带有${}号的SQL信息封装到TextSqlNode
        // ****将带有#{}号的SQL信息封装到StaticTextSqlNode
        // ****将动态SQL标签中的SQL信息分别封装到不同的SqlNode中
        MixedSqlNode rootSqlNode = parseDynamicTags(context);
        SqlSource sqlSource = null;
        // 如果SQL中包含${}和动态SQL语句，则将SqlNode封装到DynamicSqlSource
        if (isDynamic) {
            sqlSource = new DynamicSqlSource(configuration, rootSqlNode);
        } else {
            // 如果SQL中包含#{}，则将SqlNode封装到RawSqlSource中，并指定parameterType
            sqlSource = new RawSqlSource(configuration, rootSqlNode, parameterType);
        }
        return sqlSource;
    }
```



#### 1 XMLScriptBuilder#parseDynamicTags

解析select\insert\ update\delete标签中的SQL语句，最终将解析到的SqlNode封装到MixedSqlNode中的List集合中。

- 将带有${}号的SQL信息封装到TextSqlNode；
- 将带有#{}号的SQL信息封装到StaticTextSqlNode
- 将动态SQL标签中的SQL信息分别封装到不同的SqlNode中

```java
protected MixedSqlNode parseDynamicTags(XNode node) {
        List<SqlNode> contents = new ArrayList<>();
        //获取<select>\<insert>等4个标签的子节点，子节点包括元素节点和文本节点
        NodeList children = node.getNode().getChildNodes();
        for (int i = 0; i < children.getLength(); i++) {
            XNode child = node.newXNode(children.item(i));
            // 处理文本节点
            if (child.getNode().getNodeType() == Node.CDATA_SECTION_NODE
                    || child.getNode().getNodeType() == Node.TEXT_NODE) {
                String data = child.getStringBody("");
                // 将文本内容封装到SqlNode中
                TextSqlNode textSqlNode = new TextSqlNode(data);
                // SQL语句中带有${}的话，就表示是dynamic的
                if (textSqlNode.isDynamic()) {
                    contents.add(textSqlNode);
                    isDynamic = true;
                } else {
                    // SQL语句中（除了${}和下面的动态SQL标签），就表示是static的
                    // StaticTextSqlNode的apply只是进行字符串的追加操作
                    contents.add(new StaticTextSqlNode(data));
                }
                
                //处理元素节点
            } else if (child.getNode().getNodeType() == Node.ELEMENT_NODE) { // issue #628
                String nodeName = child.getNode().getNodeName();
                // 动态SQL标签处理器
                // 思考，此处使用了哪种设计模式？---策略模式
                NodeHandler handler = nodeHandlerMap.get(nodeName);
                if (handler == null) {
                    throw new BuilderException("Unknown element <" + nodeName + "> in SQL statement.");
                }
                handler.handleNode(child, contents);
                // 动态SQL标签是dynamic的
                isDynamic = true;
            }
        }
        return new MixedSqlNode(contents);
    }
```



#### 2. DynamicSqlSource#构造函数

如果SQL中包含${}和动态SQL语句，则将SqlNode封装到DynamicSqlSource

```java
public DynamicSqlSource(Configuration configuration, SqlNode rootSqlNode) {
    this.configuration = configuration;
    this.rootSqlNode = rootSqlNode;
}
```



#### 3. RawSqlSource#构造函数

如果SQL中包含#{}，则将SqlNode封装到RawSqlSource中，并指定parameterType

```java
private final SqlSource sqlSource;

    //先调用 getSql(configuration, rootSqlNode)获取sql,再走下面的构造函数
    public RawSqlSource(Configuration configuration, SqlNode rootSqlNode, Class<?> parameterType) {
        this(configuration, getSql(configuration, rootSqlNode), parameterType);
    }

    public RawSqlSource(Configuration configuration, String sql, Class<?> parameterType) {
        // 解析SQL语句
        SqlSourceBuilder sqlSourceParser = new SqlSourceBuilder(configuration);
        // 获取入参类型
        Class<?> clazz = parameterType == null ? Object.class : parameterType;
        // 开始解析
        sqlSource = sqlSourceParser.parse(sql, clazz, new HashMap<String, Object>());
    }

    private static String getSql(Configuration configuration, SqlNode rootSqlNode) {
        DynamicContext context = new DynamicContext(configuration, null);
        rootSqlNode.apply(context);
        return context.getSql();
    }
```



#### 3.1 SqlSourceBuilder#parse

解析SQL语句

```java
public SqlSource parse(String originalSql, Class<?> parameterType, Map<String, Object> additionalParameters) {
        ParameterMappingTokenHandler handler = new ParameterMappingTokenHandler(configuration, parameterType,
                additionalParameters);
        // 创建分词解析器
        GenericTokenParser parser = new GenericTokenParser("#{", "}", handler);
        // 解析#{}
        String sql = parser.parse(originalSql);
        // 将解析之后的SQL信息，封装到StaticSqlSource对象中
        // SQL字符串是带有?号的字符串，?相关的参数信息，封装到ParameterMapping集合中
        return new StaticSqlSource(configuration, sql, handler.getParameterMappings());
    }
```



#### 3.1.1 ParameterMappingTokenHandler#构造函数

```java
public ParameterMappingTokenHandler(Configuration configuration, Class<?> parameterType,Map<String, Object> additionalParameters) {
    super(configuration);
    this.parameterType = parameterType;
    this.metaParameters = configuration.newMetaObject(additionalParameters);
}
```



#### 3.1.2 GenericTokenParser#构造函数

创建分词解析器，指定待分析的openToken和closeToken，并指定处理器

```java
 public GenericTokenParser(String openToken, String closeToken, TokenHandler handler) {
    this.openToken = openToken;
    this.closeToken = closeToken;
    this.handler = handler;
  }
```



#### 3.1.3 GenericTokenParser#parse

解析SQL语句，处理openToken和closeToken中的内容

```java
/**
   * 解析${}和#{}
   * @param text
   * @return
   */
  public String parse(String text) {
    if (text == null || text.isEmpty()) {
      return "";
    }
    // search open token
    int start = text.indexOf(openToken, 0);
    if (start == -1) {
      return text;
    }
    char[] src = text.toCharArray();
    int offset = 0;
    final StringBuilder builder = new StringBuilder();
    StringBuilder expression = null;
    while (start > -1) {
      if (start > 0 && src[start - 1] == '\\') {
        // this open token is escaped. remove the backslash and continue.
        builder.append(src, offset, start - offset - 1).append(openToken);
        offset = start + openToken.length();
      } else {
        // found open token. let's search close token.
        if (expression == null) {
          expression = new StringBuilder();
        } else {
          expression.setLength(0);
        }
        builder.append(src, offset, start - offset);
        offset = start + openToken.length();
        int end = text.indexOf(closeToken, offset);
        while (end > -1) {
          if (end > offset && src[end - 1] == '\\') {
            // this close token is escaped. remove the backslash and continue.
            expression.append(src, offset, end - offset - 1).append(closeToken);
            offset = end + closeToken.length();
            end = text.indexOf(closeToken, offset);
          } else {
            expression.append(src, offset, end - offset);
            offset = end + closeToken.length();
            break;
          }
        }
        if (end == -1) {
          // close token was not found.
          builder.append(src, start, src.length - start);
          offset = src.length;
        } else {
          builder.append(handler.handleToken(expression.toString()));
          offset = end + closeToken.length();
        }
      }
      start = text.indexOf(openToken, offset);
    }
    if (offset < src.length) {
      builder.append(src, offset, src.length - offset);
    }
    return builder.toString();
  }
```

* #### 3.1.3.1 ParameterMappingTokenHandler#handleToken

处理token（#{}/${}）

```java
@Override
public String handleToken(String content) {
    parameterMappings.add(buildParameterMapping(content));
    return "?";
}
```

* #### 3.1.3.1.1 ParameterMappingTokenHandler#buildParameterMapping

创建ParameterMapping对象

```java
private ParameterMapping buildParameterMapping(String content) {
    Map<String, String> propertiesMap = parseParameterMapping(content);
    String property = propertiesMap.get("property");
    Class<?> propertyType;
    if (metaParameters.hasGetter(property)) { // issue #448 get type from additional params
        propertyType = metaParameters.getGetterType(property);
    } else if (typeHandlerRegistry.hasTypeHandler(parameterType)) {
        propertyType = parameterType;
    } else if (JdbcType.CURSOR.name().equals(propertiesMap.get("jdbcType"))) {
        propertyType = java.sql.ResultSet.class;
    } else if (property == null || Map.class.isAssignableFrom(parameterType)) {
        propertyType = Object.class;
    } else {
        MetaClass metaClass = MetaClass.forClass(parameterType, configuration.getReflectorFactory());
        if (metaClass.hasGetter(property)) {
            propertyType = metaClass.getGetterType(property);
        } else {
            propertyType = Object.class;
        }
    }
    ParameterMapping.Builder builder = new ParameterMapping.Builder(configuration, property, propertyType);
    Class<?> javaType = propertyType;
    String typeHandlerAlias = null;
    for (Map.Entry<String, String> entry : propertiesMap.entrySet()) {
        String name = entry.getKey();
        String value = entry.getValue();
        if ("javaType".equals(name)) {
            javaType = resolveClass(value);
            builder.javaType(javaType);
        } else if ("jdbcType".equals(name)) {
            builder.jdbcType(resolveJdbcType(value));
        } else if ("mode".equals(name)) {
            builder.mode(resolveParameterMode(value));
        } else if ("numericScale".equals(name)) {
            builder.numericScale(Integer.valueOf(value));
        } else if ("resultMap".equals(name)) {
            builder.resultMapId(value);
        } else if ("typeHandler".equals(name)) {
            typeHandlerAlias = value;
        } else if ("jdbcTypeName".equals(name)) {
            builder.jdbcTypeName(value);
        } else if ("property".equals(name)) {
            // Do Nothing
        } else if ("expression".equals(name)) {
            throw new BuilderException("Expression based parameters are not supported yet");
        } else {
            throw new BuilderException("An invalid property '" + name + "' was found in mapping #{" + content
                    + "}.  Valid properties are " + parameterProperties);
        }
    }
    if (typeHandlerAlias != null) {
        builder.typeHandler(resolveTypeHandler(javaType, typeHandlerAlias));
    }
    return builder.build();
}
```



#### 3.1.4 StaticSqlSource#构造函数

将解析之后的SQL信息，封装到StaticSqlSource

```java
  private final String sql;
  private final List<ParameterMapping> parameterMappings;
  private final Configuration configuration;

  public StaticSqlSource(Configuration configuration, String sql) {
    this(configuration, sql, null);
  }

  public StaticSqlSource(Configuration configuration, String sql, List<ParameterMapping> parameterMappings) {
    this.sql = sql;
    this.parameterMappings = parameterMappings;
    this.configuration = configuration;
  }
```



## 7. 源码剖析-揭秘SqlSession执行主流程

### 7.1 相关类与接口

* DefaultSqlSession：SqlSession接口的默认实现类
* Executor接口

![image-20210827153314890](.\img\image-20210827153314890.png)

* BaseExecutor：基础执行器，封装了子类的公共方法，包括一级缓存、延迟加载、回滚、关闭等功能；
* SimpleExecutor：简单执行器，每执行一条 sql，都会打开一个 Statement，执行完成后关闭；
* ReuseExecutor：重用执行器，相较于 SimpleExecutor 多了 Statement 的缓存功能，其内部维护一个 `Map<String, Statement>`，每次编译完成的 Statement 都会进行缓存，不会关闭；
* BatchExecutor：批量执行器，基于 JDBC 的 `addBatch、executeBatch` 功能，并且在当前 sql 和上一条 sql 完全一样的时候，重用 Statement，在调用 `doFlushStatements` 的时候，将数据刷新到数据库；
* CachingExecutor：缓存执行器，装饰器模式，在开启缓存的时候。会在上面三种执行器的外面包上 CachingExecutor；

* StatementHandler接口：

<img src=".\img\image-20210805193919873.png" alt="image-20210805193919873" style="zoom:150%;" />

* RoutingStatementHandler：路由。Mybatis实际使用的类，拦截的StatementHandler实际就是它。它会根据Exector类型创建对应的StatementHandler，保存到属性delegate中

* PreparedStatementHandler：预编译Statement
* ResultSetHandler接口：处理Statement执行后产生的结果集，生成结果列表；处理存储过程执行后的输出参数
* DefaultResultSetHandler：ResultSetHandler的 默认实现类



### 7.2 流程分析

### 入口：DefaultSqlSession#selectList

```java
   @Override
    public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
        try {
            // 根据传入的statementId，获取MappedStatement对象
            MappedStatement ms = configuration.getMappedStatement(statement);
            // 调用执行器的查询方法
            // RowBounds是用来逻辑分页（按照条件将数据从数据库查询到内存中，在内存中进行分页）
            // wrapCollection(parameter)是用来装饰集合或者数组参数
            return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
        } catch (Exception e) {
            throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
        } finally {
            ErrorContext.instance().reset();
        }
    }
```



#### 1. CachingExecutor#query

Configuration中cacheEnabled属性值默认为true

```java
  //第一步
  @Override
  public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    // 获取绑定的SQL语句，比如“SELECT * FROM user WHERE id = ? ” 
    BoundSql boundSql = ms.getBoundSql(parameterObject);
    // 生成缓存Key
    CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
    
    return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
  }

  //第二步
  @Override
  public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
      throws SQLException {
    // 获取二级缓存
    Cache cache = ms.getCache();
    if (cache != null) {
      // 当为select语句时，flushCache默认为false，表示任何时候语句被调用，都不会去清空本地缓存和二级缓存
      // 当为insert、update、delete语句时,useCache默认为true，表示会将本条语句的结果进行二级缓存
      // 刷新二级缓存 （存在缓存且flushCache为true时）
      flushCacheIfRequired(ms);
      if (ms.isUseCache() && resultHandler == null) {
        ensureNoOutParams(ms, boundSql);
        // 从二级缓存中查询数据
        @SuppressWarnings("unchecked")
        List<E> list = (List<E>) tcm.getObject(cache, key);
        // 如果二级缓存中没有查询到数据，则查询数据库
        if (list == null) {
          // 委托给BaseExecutor执行
          list = delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
          tcm.putObject(cache, key, list); // issue #578 and #116
        }
        return list;
      }
    }
    // 委托给BaseExecutor执行
    return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
  }
```

#### 2. BaseExecutor#query

二级缓存设置开启且缓存中没有或者未开启二级缓存，则从一级缓存中查找结果集

```java
@Override
  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    if (queryStack == 0 && ms.isFlushCacheRequired()) {
      clearLocalCache();
    }
    List<E> list;
    try {
      queryStack++;
      // 从一级缓存中获取数据
      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
      if (list != null) {
        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
      } else {
        // 如果一级缓存没有数据，则从数据库查询数据
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
      }
    } finally {
      queryStack--;
    }
    if (queryStack == 0) {
      for (DeferredLoad deferredLoad : deferredLoads) {
        deferredLoad.load();
      }
      // issue #601
      deferredLoads.clear();
      if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
        // issue #482
        clearLocalCache();
      }
    }
    return list;
  }
```



#### 3. BaseExecutor#queryFromDatabase

如果一级缓存没有数据，则从数据库查询数据

```java
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    List<E> list;
    localCache.putObject(key, EXECUTION_PLACEHOLDER);
    try {
      // 执行查询
      list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
      //移除一级缓存中原有值
      localCache.removeObject(key);
    }
    //往一级缓存中存值
    localCache.putObject(key, list);
    if (ms.getStatementType() == StatementType.CALLABLE) {
      localOutputParameterCache.putObject(key, parameter);
    }
    return list;
  }
```



#### 4. SimpleExecutor#doQuery

![image-20210831145238626](.\img\image-20210831145238626.png)

- **BaseStatementHandler**：基础语句处理器（抽象类），它基本把语句处理器接口的核心部分都实现了，包括配置绑定、执行器绑定、映射器绑定、参数处理器构建、结果集处理器构建、语句超时设置、语句关闭等，并另外定义了新的方法 instantiateStatement 供不同子类实现以便获取不同类型的语句连接，子类可以普通执行 SQL 语句，也可以做预编译执行，还可以执行存储过程等。
- **SimpleStatementHandler**：普通语句处理器，继承 BaseStatementHandler 抽象类，对应 java.sql.Statement 对象的处理，处理普通的不带动态参数运行的 SQL，即执行简单拼接的字符串语句，同时由于 Statement 的特性，SimpleStatementHandler 每次执行都需要编译 SQL （**注意：我们知道 SQL 的执行是需要编译和解析的**）。
- **PreparedStatementHandler**：预编译语句处理器，继承 BaseStatementHandler 抽象类，对应 java.sql.PrepareStatement 对象的处理，相比上面的普通语句处理器，它支持可变参数 SQL 执行，由于 PrepareStatement 的特性，它会进行预编译，在缓存中一旦发现有预编译的命令，会直接解析执行，所以减少了再次编译环节，能够有效提高系统性能，并预防 SQL 注入攻击（**所以是系统默认也是我们推荐的语句处理器**）。
- **CallableStatementHandler**：存储过程处理器，继承 BaseStatementHandler 抽象类，对应 java.sql.CallableStatement 对象的处理，很明了，它是用来调用存储过程的，增加了存储过程的函数调用以及输出/输入参数的处理支持。

- **RoutingStatementHandler**：路由语句处理器，直接实现了 StatementHandler 接口，作用如其名称，确确实实只是起到了路由功能，并把上面介绍到的三个语句处理器实例作为自身的委托对象而已，所以执行器在构建语句处理器时，都是直接 new 了 RoutingStatementHandler 实例。



执行查询

```java
    @Override
    public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler,
            BoundSql boundSql) throws SQLException {
        Statement stmt = null;
        try {
            // 获取Configuration对象
            Configuration configuration = ms.getConfiguration();
            // 创建RoutingStatementHandler，用来处理Statement
            // RoutingStatementHandler类中初始化delegate类（SimpleStatementHandler、PreparedStatementHandler）
            StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds,
                    resultHandler, boundSql);
            // 子流程1：设置参数
            stmt = prepareStatement(handler, ms.getStatementLog());
            // 子流程2：执行SQL语句（已经设置过参数），并且映射结果集
            return handler.query(stmt, resultHandler);
        } finally {
            closeStatement(stmt);
        }
    }
```



#### 4.1 Configuration#newStatementHandler

创建StatementHandler，用来执行MappedStatement对象

```java
public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement,
            Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
        // 创建路由功能的StatementHandler，根据MappedStatement中的StatementType
        StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject,
                rowBounds, resultHandler, boundSql);
        statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
        return statementHandler;
    }
```



#### 4.1.1 RoutingStatementHandler#构造函数

创建路由功能的StatementHandler，根据MappedStatement中的StatementType

```java
public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {

    switch (ms.getStatementType()) {
      case STATEMENT:
        delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      case PREPARED:
        delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      case CALLABLE:
        delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      default:
        throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
    }
  }
```



#### 4.2 SimpleExecutor#prepareStatement

设置参数

```java
 private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
        Statement stmt;
        // 获取连接
        Connection connection = getConnection(statementLog);
        // 创建Statement（PreparedStatement、Statement、CallableStatement）
        stmt = handler.prepare(connection, transaction.getTimeout());
        // SQL参数设置
        handler.parameterize(stmt);
        return stmt;
    }
```



#### 4.2.1 BaseExecutor#getConnection

获取数据库连接

```java
protected Connection getConnection(Log statementLog) throws SQLException {
    Connection connection = transaction.getConnection();
    if (statementLog.isDebugEnabled()) {
      return ConnectionLogger.newInstance(connection, statementLog, queryStack);
    } else {
      return connection;
    }
  }
```



#### 4.2.2 BaseStatementHandler#prepare

创建Statement（PreparedStatement、Statement、CallableStatement）

```java
 @Override
  public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
    ErrorContext.instance().sql(boundSql.getSql());
    Statement statement = null;
    try {
      // 实例化Statement，比如PreparedStatement
      statement = instantiateStatement(connection);
      // 设置查询超时时间
      setStatementTimeout(statement, transactionTimeout);
      setFetchSize(statement);
      return statement;
    } catch (SQLException e) {
      closeStatement(statement);
      throw e;
    } catch (Exception e) {
      closeStatement(statement);
      throw new ExecutorException("Error preparing statement.  Cause: " + e, e);
    }
  }
```



#### 4.2.2.1 PreparedStatementHandler#instantiateStatement

实例化PreparedStatement

```java
@Override
  protected Statement instantiateStatement(Connection connection) throws SQLException {
    // 获取带有占位符的SQL语句
    String sql = boundSql.getSql();
    // 处理带有主键返回的SQL
    if (mappedStatement.getKeyGenerator() instanceof Jdbc3KeyGenerator) {
      String[] keyColumnNames = mappedStatement.getKeyColumns();
      if (keyColumnNames == null) {
        return connection.prepareStatement(sql, PreparedStatement.RETURN_GENERATED_KEYS);
      } else {
        return connection.prepareStatement(sql, keyColumnNames);
      }
    } else if (mappedStatement.getResultSetType() == ResultSetType.DEFAULT) {
      return connection.prepareStatement(sql);
    } else {
      return connection.prepareStatement(sql, mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
    }
  }
```



#### 4.2.3 PreparedStatementHandler#parameterize

SQL参数设置,参数映射流程会详细分解

```java
@Override
  public void parameterize(Statement statement) throws SQLException {
    // 通过ParameterHandler处理参数
    parameterHandler.setParameters((PreparedStatement) statement);
  }
```



#### 4.3 PreparedStatementHandler#query

执行SQL语句（已经设置过参数），并且映射结果集

```java
@Override
  public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
    PreparedStatement ps = (PreparedStatement) statement;
    // 执行PreparedStatement，也就是执行SQL语句
    ps.execute();
    // 处理结果集
    return resultSetHandler.handleResultSets(ps);
  }
```



#### 4.3.1 PreparedStatement#execute

调用JDBC的api执行Statement

#### 4.3.2 DefaultResultSetHandler#handleResultSets

处理结果集 ,结果映射流程会详细分解

```java
@Override
    public List<Object> handleResultSets(Statement stmt) throws SQLException {
        ErrorContext.instance().activity("handling results").object(mappedStatement.getId());

        // <select>标签的resultMap属性，可以指定多个值，多个值之间用逗号（,）分割
        final List<Object> multipleResults = new ArrayList<>();

        int resultSetCount = 0;
        // 这里是获取第一个结果集，将传统JDBC的ResultSet包装成一个包含结果列元信息的ResultSetWrapper对象
        ResultSetWrapper rsw = getFirstResultSet(stmt);

        // 这里是获取所有要映射的ResultMap（按照逗号分割出来的）
        List<ResultMap> resultMaps = mappedStatement.getResultMaps();
        // 要映射的ResultMap的数量
        int resultMapCount = resultMaps.size();
        validateResultMapsCount(rsw, resultMapCount);
        // 循环处理每个ResultMap，从第一个开始处理
        while (rsw != null && resultMapCount > resultSetCount) {
            // 得到结果映射信息
            ResultMap resultMap = resultMaps.get(resultSetCount);
            // 处理结果集
            // 从rsw结果集参数中获取查询结果，再根据resultMap映射信息，将查询结果映射到multipleResults中
            handleResultSet(rsw, resultMap, multipleResults, null);

            rsw = getNextResultSet(stmt);
            cleanUpAfterHandlingResultSet();
            resultSetCount++;
        }

        // 对应<select>标签的resultSets属性，一般不使用该属性
        String[] resultSets = mappedStatement.getResultSets();
        if (resultSets != null) {
            while (rsw != null && resultSetCount < resultSets.length) {
                ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
                if (parentMapping != null) {
                    String nestedResultMapId = parentMapping.getNestedResultMapId();
                    ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
                    handleResultSet(rsw, resultMap, null, parentMapping);
                }
                rsw = getNextResultSet(stmt);
                cleanUpAfterHandlingResultSet();
                resultSetCount++;
            }
        }

        // 如果只有一个结果集合，则直接从多结果集中取出第一个
        return collapseSingleResultList(multipleResults);
    }
```



**四、总结**

###### 执行sqlsession：参数有两个（statementId和参数对象）

1. 根据statementId，去Configuration中的MappedStatement集合中查找
    对应的MappedStatement对象；
2. 取出MappedStatement中的SQL信息；
3. 取出MappedStatement中的statementType，用来创建Statement对象；
   - 取出MappedStatement中的Configuration对象，通过Configuration对象，获取DataSource对象，通过DataSource对象，创建Connection，通过Connection创建Statement对象。
   - 设置参数
   - 执行Statement
   - 处理结果集



## 8. 源码剖析-揭秘如何设置的参数？

### 入口：PreparedStatementHandler#parameterize方法

设置PreparedStatement的参数

```java
 @Override
  public void parameterize(Statement statement) throws SQLException {
    // 通过ParameterHandler处理参数
    parameterHandler.setParameters((PreparedStatement) statement);
  }
```

#### DefaultParameterHandler#setParameters 

设置参数

```java
      @Override
    public void setParameters(PreparedStatement ps) {
        ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
        // 获取要设置的参数映射信息
        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
        if (parameterMappings != null) {
            for (int i = 0; i < parameterMappings.size(); i++) {
                ParameterMapping parameterMapping = parameterMappings.get(i);
                // 只处理入参
                if (parameterMapping.getMode() != ParameterMode.OUT) {
                    Object value;
                    // 获取属性名称
                    String propertyName = parameterMapping.getProperty();
                    if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
                        value = boundSql.getAdditionalParameter(propertyName);
                    } else if (parameterObject == null) {
                        value = null;
                    } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                        value = parameterObject;
                    } else {
                        MetaObject metaObject = configuration.newMetaObject(parameterObject);
                        value = metaObject.getValue(propertyName);
                    }
                    // 获取每个参数的类型处理器，去设置入参和获取返回值
                    TypeHandler typeHandler = parameterMapping.getTypeHandler();
                    // 获取每个参数的JdbcType
                    JdbcType jdbcType = parameterMapping.getJdbcType();
                    if (value == null && jdbcType == null) {
                        jdbcType = configuration.getJdbcTypeForNull();
                    }
                    try {
                        // 给PreparedStatement设置参数
                        typeHandler.setParameter(ps, i + 1, value, jdbcType);
                    } catch (TypeException e) {
                        throw new TypeException(
                                "Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
                    } catch (SQLException e) {
                        throw new TypeException(
                                "Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
                    }
                }
            }
        }
    }
```



#### BaseTypeHandler#setParameter

给PreparedStatement设置参数

```java
@Override
    public void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {
        if (parameter == null) {
            if (jdbcType == null) {
                throw new TypeException(
                        "JDBC requires that the JdbcType must be specified for all nullable parameters.");
            }
            try {
                ps.setNull(i, jdbcType.TYPE_CODE);
            } catch (SQLException e) {
                throw new TypeException("Error setting null for parameter #" + i + " with JdbcType " + jdbcType + " . "
                        + "Try setting a different JdbcType for this parameter or a different jdbcTypeForNull configuration property. "
                        + "Cause: " + e, e);
            }
        } else {
            try {
                // 通过PreparedStatement的API去设置非空参数
                setNonNullParameter(ps, i, parameter, jdbcType);
            } catch (Exception e) {
                throw new TypeException("Error setting non null for parameter #" + i + " with JdbcType " + jdbcType
                        + " . "
                        + "Try setting a different JdbcType for this parameter or a different configuration property. "
                        + "Cause: " + e, e);
            }
        }
    }
```

#### xxxTypeHandler#setNonNullParameter

通过PreparedStatement的API去设置非空参数
例如：ArrayTypeHandler

```java
@Override
  public void setNonNullParameter(PreparedStatement ps, int i, Object parameter, JdbcType jdbcType) throws SQLException {
    ps.setArray(i, (Array) parameter);
  }
```





## 9. 源码剖析-结果集映射流程

### 入口：DefaultResultSetHandler#handleResultSets

从rsw结果集参数中获取查询结果，再根据resultMap映射信息，将查询结果映射到multipleResults中

```java
@Override
    public List<Object> handleResultSets(Statement stmt) throws SQLException {
        ErrorContext.instance().activity("handling results").object(mappedStatement.getId());

        // <select>标签的resultMap属性，可以指定多个值，多个值之间用逗号（,）分割
        final List<Object> multipleResults = new ArrayList<>();

        int resultSetCount = 0;
        // 这里是获取第一个结果集，将传统JDBC的ResultSet包装成一个包含结果列元信息的ResultSetWrapper对象
        ResultSetWrapper rsw = getFirstResultSet(stmt);

        // 这里是获取所有要映射的ResultMap（按照逗号分割出来的）
        List<ResultMap> resultMaps = mappedStatement.getResultMaps();
        // 要映射的ResultMap的数量
        int resultMapCount = resultMaps.size();
        validateResultMapsCount(rsw, resultMapCount);
        // 循环处理每个ResultMap，从第一个开始处理
        while (rsw != null && resultMapCount > resultSetCount) {
            // 得到结果映射信息
            ResultMap resultMap = resultMaps.get(resultSetCount);
            // 处理结果集
            // 从rsw结果集参数中获取查询结果，再根据resultMap映射信息，将查询结果映射到multipleResults中
            handleResultSet(rsw, resultMap, multipleResults, null);

            rsw = getNextResultSet(stmt);
            cleanUpAfterHandlingResultSet();
            resultSetCount++;
        }

        // 对应<select>标签的resultSets属性，一般不使用该属性
        String[] resultSets = mappedStatement.getResultSets();
        if (resultSets != null) {
            while (rsw != null && resultSetCount < resultSets.length) {
                ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
                if (parentMapping != null) {
                    String nestedResultMapId = parentMapping.getNestedResultMapId();
                    ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
                    handleResultSet(rsw, resultMap, null, parentMapping);
                }
                rsw = getNextResultSet(stmt);
                cleanUpAfterHandlingResultSet();
                resultSetCount++;
            }
        }

        // 如果只有一个结果集合，则直接从多结果集中取出第一个
        return collapseSingleResultList(multipleResults);
    }
```



#### DefaultResultSetHandler#handleRowValues

处理行数据，其实就是完成结果映射

```java
public void handleRowValues(ResultSetWrapper rsw, ResultMap resultMap, ResultHandler<?> resultHandler,
            RowBounds rowBounds, ResultMapping parentMapping) throws SQLException {
        // 是否有内置嵌套的结果映射
        if (resultMap.hasNestedResultMaps()) {
            ensureNoRowBounds();
            checkResultHandler();
            // 嵌套结果映射
            handleRowValuesForNestedResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
        } else {
            // 简单结果映射
            handleRowValuesForSimpleResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
        }
    }
```



#### DefaultResultSetHandler#handleRowValuesForSimpleResultMap

简单结果映射

```java
private void handleRowValuesForSimpleResultMap(ResultSetWrapper rsw, ResultMap resultMap,
            ResultHandler<?> resultHandler, RowBounds rowBounds, ResultMapping parentMapping) throws SQLException {
        DefaultResultContext<Object> resultContext = new DefaultResultContext<>();
        // 获取结果集信息
        ResultSet resultSet = rsw.getResultSet();
        // 使用rowBounds的分页信息，进行逻辑分页（也就是在内存中分页）
        skipRows(resultSet, rowBounds);
        while (shouldProcessMoreRows(resultContext, rowBounds) && !resultSet.isClosed() && resultSet.next()) {
            // 通过<resultMap>标签的子标签<discriminator>对结果映射进行鉴别
            ResultMap discriminatedResultMap = resolveDiscriminatedResultMap(resultSet, resultMap, null);
            // 将查询结果封装到POJO中
            Object rowValue = getRowValue(rsw, discriminatedResultMap, null);
            // 处理对象嵌套的映射关系
            storeObject(resultHandler, resultContext, rowValue, parentMapping, resultSet);
        }
    }
```



#### 1. DefaultResultSetHandler#getRowValue

将查询结果封装到POJO中

```java
private Object getRowValue(ResultSetWrapper rsw, ResultMap resultMap, String columnPrefix) throws SQLException {
        // 延迟加载的映射信息
        final ResultLoaderMap lazyLoader = new ResultLoaderMap();
        // 创建要映射的PO类对象
        Object rowValue = createResultObject(rsw, resultMap, lazyLoader, columnPrefix);
        if (rowValue != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
            final MetaObject metaObject = configuration.newMetaObject(rowValue);
            boolean foundValues = this.useConstructorMappings;
            // 是否应用自动映射，也就是通过resultType进行映射
            if (shouldApplyAutomaticMappings(resultMap, false)) {
                // 根据columnName和type属性名映射赋值
                foundValues = applyAutomaticMappings(rsw, resultMap, metaObject, columnPrefix) || foundValues;
            }
            // 根据我们配置ResultMap的column和property映射赋值
            // 如果映射存在nestedQueryId，会调用getNestedQueryMappingValue方法获取返回值
            foundValues = applyPropertyMappings(rsw, resultMap, metaObject, lazyLoader, columnPrefix) || foundValues;
            foundValues = lazyLoader.size() > 0 || foundValues;
            rowValue = foundValues || configuration.isReturnInstanceForEmptyRow() ? rowValue : null;
        }
        return rowValue;
    }
```



#### 1.1 DefaultResultSetHandler#createResultObject

创建映射结果对象

```java
private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, ResultLoaderMap lazyLoader,
            String columnPrefix) throws SQLException {
        this.useConstructorMappings = false; // reset previous mapping result
        final List<Class<?>> constructorArgTypes = new ArrayList<>();
        final List<Object> constructorArgs = new ArrayList<>();
        // 创建结果映射的PO类对象
        Object resultObject = createResultObject(rsw, resultMap, constructorArgTypes, constructorArgs, columnPrefix);
        if (resultObject != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
            // 获取要映射的PO类的属性信息
            final List<ResultMapping> propertyMappings = resultMap.getPropertyResultMappings();
            for (ResultMapping propertyMapping : propertyMappings) {
                // issue gcode #109 && issue #149
                // 延迟加载处理
                if (propertyMapping.getNestedQueryId() != null && propertyMapping.isLazy()) {
                    // 通过动态代理工厂，创建延迟加载的代理对象
                    resultObject = configuration.getProxyFactory().createProxy(resultObject, lazyLoader, configuration,
                            objectFactory, constructorArgTypes, constructorArgs);
                    break;
                }
            }
        }
        this.useConstructorMappings = resultObject != null && !constructorArgTypes.isEmpty(); // set current mapping
                                                                                                // result
        return resultObject;
    }

private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, List<Class<?>> constructorArgTypes,
            List<Object> constructorArgs, String columnPrefix) throws SQLException {
        final Class<?> resultType = resultMap.getType();
        final MetaClass metaType = MetaClass.forClass(resultType, reflectorFactory);
        final List<ResultMapping> constructorMappings = resultMap.getConstructorResultMappings();
        if (hasTypeHandlerForResultObject(rsw, resultType)) {
            return createPrimitiveResultObject(rsw, resultMap, columnPrefix);
        } else if (!constructorMappings.isEmpty()) {
            return createParameterizedResultObject(rsw, resultType, constructorMappings, constructorArgTypes,
                    constructorArgs, columnPrefix);
        } else if (resultType.isInterface() || metaType.hasDefaultConstructor()) {
            // 对象工厂创建对象
            return objectFactory.create(resultType);
        } else if (shouldApplyAutomaticMappings(resultMap, false)) {
            return createByConstructorSignature(rsw, resultType, constructorArgTypes, constructorArgs, columnPrefix);
        }
        throw new ExecutorException("Do not know how to create an instance of " + resultType);
    }
```



#### 1.2 DefaultResultSetHandler#applyAutomaticMappings

根据columnName和type属性名映射赋值

```java
private boolean applyAutomaticMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject,
            String columnPrefix) throws SQLException {
        List<UnMappedColumnAutoMapping> autoMapping = createAutomaticMappings(rsw, resultMap, metaObject, columnPrefix);
        boolean foundValues = false;
        if (!autoMapping.isEmpty()) {
            for (UnMappedColumnAutoMapping mapping : autoMapping) {
                final Object value = mapping.typeHandler.getResult(rsw.getResultSet(), mapping.column);
                if (value != null) {
                    foundValues = true;
                }
                if (value != null || (configuration.isCallSettersOnNulls() && !mapping.primitive)) {
                    // gcode issue #377, call setter on nulls (value is not 'found')
                    metaObject.setValue(mapping.property, value);
                }
            }
        }
        return foundValues;
    }
```

#### 1.3 DefaultResultSetHandler#applyPropertyMappings

根据我们配置ResultMap的column和property映射赋值,如果映射存在nestedQueryId，会调用getNestedQueryMappingValue方法获取返回值

```java
private boolean applyPropertyMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject,
            ResultLoaderMap lazyLoader, String columnPrefix) throws SQLException {
        final List<String> mappedColumnNames = rsw.getMappedColumnNames(resultMap, columnPrefix);
        boolean foundValues = false;
        final List<ResultMapping> propertyMappings = resultMap.getPropertyResultMappings();
        for (ResultMapping propertyMapping : propertyMappings) {
            String column = prependPrefix(propertyMapping.getColumn(), columnPrefix);
            if (propertyMapping.getNestedResultMapId() != null) {
                // the user added a column attribute to a nested result map, ignore it
                column = null;
            }
            if (propertyMapping.isCompositeResult()
                    || (column != null && mappedColumnNames.contains(column.toUpperCase(Locale.ENGLISH)))
                    || propertyMapping.getResultSet() != null) {
                Object value = getPropertyMappingValue(rsw.getResultSet(), metaObject, propertyMapping, lazyLoader,
                        columnPrefix);
                // issue #541 make property optional
                final String property = propertyMapping.getProperty();
                if (property == null) {
                    continue;
                } else if (value == DEFERRED) {
                    foundValues = true;
                    continue;
                }
                if (value != null) {
                    foundValues = true;
                }
                if (value != null || (configuration.isCallSettersOnNulls()
                        && !metaObject.getSetterType(property).isPrimitive())) {
                    // gcode issue #377, call setter on nulls (value is not 'found')
                    metaObject.setValue(property, value);
                }
            }
        }
        return foundValues;
    }
```



## 10. 源码剖析-获取Mapper代理对象流程

### 入口：DefaultSqlSession#getMapper

从Configuration对象中，根据Mapper接口，获取Mapper代理对象

```java
    @Override
    public <T> T getMapper(Class<T> type) {
        // 从Configuration对象中，根据Mapper接口，获取Mapper代理对象
        return configuration.<T>getMapper(type, this);
    }
```



#### Configuration#getMapper

获取Mapper代理对象

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return mapperRegistry.getMapper(type, sqlSession);
}
```



#### 1. MapperRegistry#getMapper

通过代理对象工厂，获取代理对象：

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    // 根据Mapper接口的类型，从Map集合中获取Mapper代理对象工厂
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      // 通过MapperProxyFactory生产MapperProxy，通过MapperProxy产生Mapper代理对象
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
  }
```



#### 1.1 MapperProxyFactory#newInstance

调用JDK的动态代理方式，创建Mapper代理

```java
  //1
 protected T newInstance(MapperProxy<T> mapperProxy) {
    // 使用JDK动态代理方式，生成代理对象
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }

//2
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    // 根据Mapper接口的类型，从Map集合中获取Mapper代理对象工厂
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      // 通过MapperProxyFactory生产MapperProxy，通过MapperProxy产生Mapper代理对象
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
  }
```



## 11. 源码剖析-invoke方法

```java
// 通过JDK动态代理生成并获取代理对象       
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
// 代理对象对象调用方法，底层执行invoke方法       
List<User> allUser = userMapper.findAllUser();
```

在动态代理返回了示例后，我们就可以直接调用mapper类中的方法了，但**代理对象调用方法，执行是在MapperProxy中的invoke方法**，该类实现**InvocationHandler**接口，并重写invoke()方法。

问题：invoke方法执行逻辑是什么？



### 入口：**MapperProxy**#invoke

```java
 @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            // 如果是 Object 定义的方法，直接调用
            if (Object.class.equals(method.getDeclaringClass())) {
                return method.invoke(this, args);

            } else if (isDefaultMethod(method)) {
                return invokeDefaultMethod(proxy, method, args);
            }
        } catch (Throwable t) {
            throw ExceptionUtil.unwrapThrowable(t);
        }
        // 获得 MapperMethod 对象
        final MapperMethod mapperMethod = cachedMapperMethod(method);
        // 重点在这：MapperMethod最终调用了执行的方法
        return mapperMethod.execute(sqlSession, args);
    }
```

#### **MapperMethod**

```java
public Object execute(SqlSession sqlSession, Object[] args) {
        Object result;
        //判断mapper中的方法类型，最终调用的还是SqlSession中的方法
        switch (command.getType()) {
            case INSERT: {
                // 转换参数
                Object param = method.convertArgsToSqlCommandParam(args);
                // 执行 INSERT 操作
                // 转换 rowCount
                result = rowCountResult(sqlSession.insert(command.getName(), param));
                break;
            }
            case UPDATE: {
                // 转换参数
                Object param = method.convertArgsToSqlCommandParam(args);
                // 转换 rowCount
                result = rowCountResult(sqlSession.update(command.getName(), param));
                break;
            }
            case DELETE: {
                // 转换参数
                Object param = method.convertArgsToSqlCommandParam(args);
                // 转换 rowCount
                result = rowCountResult(sqlSession.delete(command.getName(), param));
                break;
            }
            case SELECT:
                // 无返回，并且有 ResultHandler 方法参数，则将查询的结果，提交给 ResultHandler 进行处理
                if (method.returnsVoid() && method.hasResultHandler()) {
                    executeWithResultHandler(sqlSession, args);
                    result = null;
                // 执行查询，返回列表
                } else if (method.returnsMany()) {
                    result = executeForMany(sqlSession, args);
                // 执行查询，返回 Map
                } else if (method.returnsMap()) {
                    result = executeForMap(sqlSession, args);
                // 执行查询，返回 Cursor
                } else if (method.returnsCursor()) {
                    result = executeForCursor(sqlSession, args);
                // 执行查询，返回单个对象
                } else {
                    // 转换参数
                    Object param = method.convertArgsToSqlCommandParam(args);
                    // 查询单条
                    result = sqlSession.selectOne(command.getName(), param);
                    if (method.returnsOptional() &&
                            (result == null || !method.getReturnType().equals(result.getClass()))) {
                        result = Optional.ofNullable(result);
                    }
                }
                break;
            case FLUSH:
                result = sqlSession.flushStatements();
                break;
            default:
                throw new BindingException("Unknown execution method for: " + command.getName());
        }
        // 返回结果为 null ，并且返回类型为基本类型，则抛出 BindingException 异常
        if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
            throw new BindingException("Mapper method '" + command.getName()
                    + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
        }
        // 返回结果
        return result;
    }
```



## 12. 源码剖析-插件机制

### 12.1  插件概述

<img src=".\img\image-20210809154420521.png" alt="image-20210809154420521" style="zoom:80%;" />

* 问题：什么是Mybatis插件？有什么作用？

一般开源框架都会提供扩展点，让开发者自行扩展，从而完成逻辑的增强。

基于插件机制可以实现了很多有用的功能，比如说分页，字段加密，监控等功能，这种通用的功能，就如同AOP一样，横切在数据操作上

**而通过Mybatis插件可以实现对框架的扩展，来实现自定义功能，并且对于用户是无感知的。**



### 12.2 Mybatis插件介绍

Mybatis插件本质上来说就是一个拦截器，它体现了`JDK动态代理和责任链设计模式的综合运用`

Mybatis中针对四大组件提供了扩展机制，这四个组件分别是：

<img src=".\img\image-20210903144451327.png" alt="image-20210903144451327" style="zoom:80%;" />

Mybatis中所允许拦截的方法如下：

- Executor 【SQL执行器】【update,query,commit,rollback】
- StatementHandler 【Sql语法构建器对象】【prepare,parameterize,batch,update,query等】
- ParameterHandler 【参数处理器】【getParameterObject,setParameters等】
- ResultSetHandler 【结果集处理器】【handleResultSets,handleOuputParameters等】



#### 能干什么？

* 分页功能：mybatis的分页默认是基于内存分页的（查出所有，再截取），数据量大的情况下效率较低，不过使用mybatis插件可以改变该行为，只需要拦截StatementHandler类的prepare方法，改变要执行的SQL语句为分页语句即可

* 性能监控：对于SQL语句执行的性能监控，可以通过拦截Executor类的update, query等方法，用日志记录每个方法执行的时间



#### 如何自定义插件？

在使用之前，我们先来看看Mybatis提供的插件相关的类，过一遍它们分别提供了哪些功能，最后我们自己定义一个插件

**用于定义插件的类**

前面已经知道Mybatis插件是可以对Mybatis中四大组件对象的方法进行拦截，那拦截器拦截哪个类的哪个方法如何知道，就由下面这个注解提供拦截信息

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Intercepts {  
  Signature[] value();
}
```



由于一个拦截器可以同时拦截多个对象的多个方法，所以就使用了`Signture`数组，该注解定义了拦截的完整信息

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({})
public @interface Signature {
  // 拦截的类
  Class<?> type();
  // 拦截的方法
  String method();
  // 拦截方法的参数    
  Class<?>[] args();
  
 } 
```



已经知道了该拦截哪些对象的哪些方法，拦截后要干什么就需要实现`Intercetor#intercept`方法，在这个方法里面实现拦截后的处理逻辑

```java

public interface Interceptor {
  /**
   * 真正方法被拦截执行的逻辑
   *
   * @param invocation 主要目的是将多个参数进行封装
   */
  Object intercept(Invocation invocation) throws Throwable;
    
  // 生成目标对象的代理对象
  default Object plugin(Object target) {
    return Plugin.wrap(target, this);
  }
  // 可以拦截器设置一些属性
  default void setProperties(Properties properties) {
    // NOP
  }
}
```



### 12.3 自定义插件

**需求**：把Mybatis所有执行的sql都记录下来

**步骤**：①  创建Interceptor的实现类，重写方法

​			②  使用@Intercepts注解完成插件签名 说明插件的拦截四大对象之一的哪一个对象的哪一个方法

​            ③  将写好的插件注册到全局配置文件中



①.创建Interceptor的实现类

```java
public class MyPlugin implements Interceptor {
	private final Logger logger = LoggerFactory.getLogger(this.getClass());
	// //这里是每次执行操作的时候，都会进行这个拦截器的方法内 
	
	Override
	public Object intercept(Invocation invocation) throws Throwable { 
	//增强逻辑
	 StatementHandler statementHandler = (StatementHandler) invocation.getTarget();
        BoundSql boundSql = statementHandler.getBoundSql();
        String sql = boundSql.getSql();
        logger.info("mybatis intercept sql:{}", sql);
		return invocation.proceed(); //执行原方法 
} 
	
	/**
	*
	*	^Description包装目标对象 为目标对象创建代理对象
	*	@Param target为要拦截的对象
	*	@Return代理对象
	*/
	Override 
	public Object plugin(Object target) {
		System.out.println("将要包装的目标对象："+target); 
		return Plugin.wrap(target,this);
    }
    
	/**获取配置文件的属性**/
	//插件初始化的时候调用，也只调用一次，插件配置的属性从这里设置进来
	Override
    public void setProperties(Properties properties) {
		System.out.println("插件配置的初始化参数："+properties );
	}
}
```

②  使用@Intercepts注解完成插件签名 说明插件的拦截四大对象之一的哪一个对象的哪一个方法

```java
@Intercepts({ @Signature(type = StatementHandler.class, 
                         method = "prepare", 
                         args = { Connection.class, Integer.class}) })
public class SQLStatsInterceptor implements Interceptor {
```

③  将写好的插件注册到全局配置文件中

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <plugins>
        <plugin interceptor="com.itheima.interceptor.MyPlugin">
            <property name="dialect" value="mysql" />
        </plugin>
    </plugins>
</configuration>
```



#### 核心思想：

就是使用JDK动态代理的方式，对这四个对象进行包装增强。具体的做法是，创建一个类实现Mybatis的拦截器接口，并且加入到拦截器链中，在创建核心对象的时候，不直接返回，而是遍历拦截器链，把每一个拦截器都作用于核心对象中。这么一来，Mybatis创建的核心对象其实都是代理对象，都是被包装过的。

<img src=".\img\image-20210809165203214.png" alt="image-20210809165203214" style="zoom:67%;" />



### 12.4 源码分析-插件 

* 插件的初始化：插件对象是如何实例化的？ 插件的实例对象如何添加到拦截器链中的？ 组件对象的代理对象是如何产生的？
* 拦截逻辑的执行



##### **插件配置信息的加载**

我们定义好了一个拦截器，那我们怎么告诉Mybatis呢？Mybatis所有的配置都定义在XXx.xml配置文件中

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <plugins>
        <plugin interceptor="com.itheima.interceptor.MyPlugin">
            <property name="dialect" value="mysql" />
        </plugin>
    </plugins>
</configuration>
```

对应的解析代码如下（XMLConfigBuilder#pluginElement）：

```java
private void pluginElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        // 获取拦截器
        String interceptor = child.getStringAttribute("interceptor");
        // 获取配置的Properties属性
        Properties properties = child.getChildrenAsProperties();
        // 根据配置文件中配置的插件类的全限定名 进行反射初始化
        Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).getDeclaredConstructor().newInstance();
        // 将属性添加到Intercepetor对象
        interceptorInstance.setProperties(properties);
        // 添加到配置类的InterceptorChain属性，InterceptorChain类维护了一个List<Interceptor>
        configuration.addInterceptor(interceptorInstance);
      }
    }
  }
```

主要做了以下工作：

1. 遍历解析plugins标签下每个plugin标签
2. 根据解析的类信息创建Interceptor对象
3. 调用setProperties方法设置属性
4. 将拦截器添加到Configuration类的IntercrptorChain拦截器链中

对应时序图如下：

<img src=".\img\image-20210810112820196.png" alt="image-20210810112820196" style="zoom:80%;" />



##### 代理对象的生成

Executor代理对象（Configuration#newExecutor）

```java

public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
      executor = new CachingExecutor(executor);
    }
    // 生成Executor代理对象逻辑
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
  }
```

ParameterHandler代理对象（Configuration#newParameterHandler）

```java
public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
    ParameterHandler parameterHandler = mappedStatement.getLang().createParameterHandler(mappedStatement, parameterObject, boundSql);
    // 生成ParameterHandler代理对象逻辑 
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
    return parameterHandler;
  }
```

ResultSetHandler代理对象（Configuration#newResultSetHandler）

```java

public ResultSetHandler newResultSetHandler(Executor executor, MappedStatement mappedStatement, RowBounds rowBounds, ParameterHandler parameterHandler,
      ResultHandler resultHandler, BoundSql boundSql) {
    ResultSetHandler resultSetHandler = new DefaultResultSetHandler(executor, mappedStatement, parameterHandler, resultHandler, boundSql, rowBounds);
    // 生成ResultSetHandler代理对象逻辑
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    return resultSetHandler;
  }
```

StatementHandler代理对象（Configuration#newStatementHandler）

```java
public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    // 生成StatementHandler代理对象逻辑
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;

}
```

通过查看源码会发现，所有代理对象的生成都是通过`InterceptorChain#pluginAll`方法来创建的，进一步查看pluginAll方法

```java

public Object pluginAll(Object target) {
    for (Interceptor interceptor : interceptors) {
        target = interceptor.plugin(target);
    }
    return target;

}
```

InterceptorChain#pluginAll内部通过遍历Interceptor#plugin方法来创建代理对象，并将生成的代理对象又赋值给target,如果存在多个拦截器的话，生成的代理对象会被另一个代理对象所代理，从而形成一个代理链，执行的时候，依次执行所有拦截器的拦截逻辑代码，我们再跟进去

```java

default Object plugin(Object target) {
  return Plugin.wrap(target, this);
}
```



`Interceptor#plugin`方法最终将目标对象和当前的拦截器交给`Plugin.wrap`方法来创建代理对象。该方法是默认方法，是Mybatis框架提供的一个典型plugin方法的实现。让我们看看在`Plugin#wrap`方法中是如何实现代理对象的

```java
public static Object wrap(Object target, Interceptor interceptor) {
    // 1.解析该拦截器所拦截的所有接口及对应拦截接口的方法
    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
    Class<?> type = target.getClass();
    // 2.获取目标对象实现的所有被拦截的接口
    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
    // 3.目标对象有实现被拦截的接口，生成代理对象并返回
    if (interfaces.length > 0) {
        // 通过JDK动态代理的方式实现
      return Proxy.newProxyInstance(
          type.getClassLoader(),
          interfaces,
          new Plugin(target, interceptor, signatureMap));
    }
    // 目标对象没有实现被拦截的接口，直接返回原对象
    return target;

}
```

最终我们看到其实是通过JDK提供的Proxy.newProxyInstance方法来生成代理对象

以上代理对象生成过程的时序图如下:

![](.\img\111.png)



##### 拦截逻辑的执行

通过上面的分析，我们知道Mybatis框架中执行Executor、ParameterHandler、ResultSetHandler和StatementHandler中的方法时真正执行的是代理对象对应的方法。而且该代理对象是通过JDK动态代理生成的，所以执行方法时实际上是调用InvocationHandler#invoke方法（Plugin类实现InvocationHandler接口）,下面是Plugin#invoke方法

```java
@Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      Set<Method> methods = signatureMap.get(method.getDeclaringClass());
      if (methods != null && methods.contains(method)) {
        return interceptor.intercept(new Invocation(target, method, args));
      }
      return method.invoke(target, args);
    } catch (Exception e) {
      throw ExceptionUtil.unwrapThrowable(e);
    }
}
```



**注：一个对象被代理很多次**

问题：同一个组件对象的同一个方法是否可以被多个拦截器进行拦截？

答案是肯定的，所以我们**配置在最前面的拦截器最先被代理，但是执行的时候却是最外层的先执行**。

具体点：

假如依次定义了三个插件：`插件1`，`插件2 和 插件3`。

那么List中就会按顺序存储：`插件1`，`插件2` 和 `插件3`。

而解析的时候是遍历list，所以解析的时候也是按照：`插件1 `,`插件2`,`插件3`的顺序。

但是执行的时候就要反过来了，执行的时候是按照：`插件3`，`插件2`和`插件1`的顺序进行执行。

<img src=".\img\image-20210903145021790.png" alt="image-20210903145021790" style="zoom:67%;" />

当 Executor 的某个方法被调用的时候，插件逻辑会先行执行。执行顺序由外而内，比如上图的执行顺序为 `plugin3 → plugin2 → Plugin1 → Executor`。



## 13. 源码剖析-缓存策略

缓存就是内存中的数据，常常来自对数据库查询结果的保存。使用缓存，我们可以避免频繁的与数据库进行交互，进而提高响应速度MyBatis也提供了对缓存的支持，分为一级缓存和二级缓存，可以通过下图来理解：

<img src=".\img\image-20210906153547734.png" alt="image-20210906153547734" style="zoom:80%;" />

①、一级缓存是SqlSession级别的缓存。在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。

②、二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的



### 一级缓存

默认是开启的

①、我们使用同一个sqlSession，对User表根据相同id进行两次查询，查看他们发出sql语句的情况

```java
  @Test
  public void firstLevelCacheTest() throws IOException {

    // 1. 通过类加载器对配置文件进行加载，加载成了字节输入流，存到内存中 注意：配置文件并没有被解析
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");

    // 2. (1)解析了配置文件，封装configuration对象 （2）创建了DefaultSqlSessionFactory工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);

    // 3.问题：openSession()执行逻辑是什么？
    // 3. (1)创建事务对象 （2）创建了执行器对象cachingExecutor (3)创建了DefaultSqlSession对象
    SqlSession sqlSession = sqlSessionFactory.openSession();

    // 4. 委派给Executor来执行，Executor执行时又会调用很多其他组件（参数设置、解析sql的获取，sql的执行、结果集的封装）
    User user = sqlSession.selectOne("com.itheima.mapper.UserMapper.findByCondition", 1);
    User user2 = sqlSession.selectOne("com.itheima.mapper.UserMapper.findByCondition", 1);

    System.out.println(user == user2);

    sqlSession.close();

  }
```

查看控制台打印情况：

<img src=".\img\image-20210906152908712.png" alt="image-20210906152908712" style="zoom:67%;" />



②  同样是对user表进行两次查询，只不过两次查询之间进行了一次update操作。

```java
  @Test
  public void test3() throws IOException {

    // 1. 通过类加载器对配置文件进行加载，加载成了字节输入流，存到内存中 注意：配置文件并没有被解析
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");

    // 2. (1)解析了配置文件，封装configuration对象 （2）创建了DefaultSqlSessionFactory工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);

    // 3.问题：openSession()执行逻辑是什么？
    // 3. (1)创建事务对象 （2）创建了执行器对象cachingExecutor (3)创建了DefaultSqlSession对象
    SqlSession sqlSession = sqlSessionFactory.openSession();

    // 4. 委派给Executor来执行，Executor执行时又会调用很多其他组件（参数设置、解析sql的获取，sql的执行、结果集的封装）
    User user = sqlSession.selectOne("com.itheima.mapper.UserMapper.findByCondition", 1);

    User user1 = new User();
    user1.setId(1);
    user1.setUsername("zimu");
    sqlSession.update("com.itheima.mapper.UserMapper.updateUser",user1);
    sqlSession.commit();
    User user2 = sqlSession.selectOne("com.itheima.mapper.UserMapper.findByCondition", 1);

    System.out.println(user == user2);
    System.out.println(user);
    System.out.println(user2);
    System.out.println("MyBatis源码环境搭建成功....");

    sqlSession.close();

  }
```

查看控制台打印情况：

<img src=".\img\image-20210906153152029.png" alt="image-20210906153152029" style="zoom:67%;" />



③、**总结**

1、第一次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，如果没有，从 数据库查询用户信息。得到用户信息，将用户信息存储到一级缓存中。

2、 如果中间sqlSession去执行commit操作（执行插入、更新、删除），则会清空SqlSession中的 一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。

3、 第二次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，缓存中有，直 接从缓存中获取用户信息



#### 一级缓存原理探究与源码分析

问题1：一级缓存 底层数据结构到底是什么？

问题2：一级缓存的工作流程是怎样的？



#### 1. 一级缓存 底层数据结构到底是什么？

之前说`不同SqlSession的一级缓存互不影响`,所以我从SqlSession这个类入手

<img src=".\img\image-20210810152249915.png" alt="image-20210810152249915" style="zoom:80%;" />

可以看到，`org.apache.ibatis.session.SqlSession`中有一个和缓存有关的方法——`clearCache()`刷新缓存的方法，点进去，找到它的实现类`DefaultSqlSession`

```java
  @Override
  public void clearCache() {
    executor.clearLocalCache();
  }
```

再次点进去`executor.clearLocalCache()`,再次点进去并找到其实现类`BaseExecutor`，

```java
  @Override
  public void clearLocalCache() {
    if (!closed) {
      localCache.clear();
      localOutputParameterCache.clear();
    }
  
```

进入`localCache.clear()`方法。进入到了`org.apache.ibatis.cache.impl.PerpetualCache`类中

```java
package org.apache.ibatis.cache.impl;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import org.apache.ibatis.cache.Cache;
import org.apache.ibatis.cache.CacheException;
/**
 * @author Clinton Begin
 */
public class PerpetualCache implements Cache {
  private final String id;

  private Map<Object, Object> cache = new HashMap<Object, Object>();

  public PerpetualCache(String id) {
    this.id = id;
  }

  //省略部分...
  @Override
  public void clear() {
    cache.clear();
  }
  //省略部分...
}

```

我们看到了`PerpetualCache`类中有一个属性`private Map<Object, Object> cache = new HashMap<Object, Object>()`，很明显它是一个HashMap，我们所调用的`.clear()`方法，**实际上就是调用的Map的clear方法**

<img src=".\img\image-20210810170739986.png" alt="image-20210810170739986" style="zoom:80%;" />



**得出结论：**

一级缓存的数据结构确实是HashMap

​                               <img src=".\img\image-20210810170835654.png" alt="image-20210810170835654" style="zoom:80%;" />



#### 2. 一级缓存的执行流程 

我们进入到`org.apache.ibatis.executor.Executor`中
看到一个方法`CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql)` ，见名思意是一个创建CacheKey的方法
找到它的实现类和方法`org.apache.ibatis.executor.BaseExecuto.createCacheKey`

<img src=".\img\image-20210810172319796.png" alt="image-20210810172319796" style="zoom:67%;" />



我们分析一下创建CacheKey的这块代码：

```java
public CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql) {
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    //初始化CacheKey
    CacheKey cacheKey = new CacheKey();
    //存入statementId
    cacheKey.update(ms.getId());
    //分别存入分页需要的Offset和Limit
    cacheKey.update(rowBounds.getOffset());
    cacheKey.update(rowBounds.getLimit());
    //把从BoundSql中封装的sql取出并存入到cacheKey对象中
    cacheKey.update(boundSql.getSql());
    //下面这一块就是封装参数
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    TypeHandlerRegistry typeHandlerRegistry = ms.getConfiguration().getTypeHandlerRegistry();

    for (ParameterMapping parameterMapping : parameterMappings) {
      if (parameterMapping.getMode() != ParameterMode.OUT) {
        Object value;
        String propertyName = parameterMapping.getProperty();
        if (boundSql.hasAdditionalParameter(propertyName)) {
          value = boundSql.getAdditionalParameter(propertyName);
        } else if (parameterObject == null) {
          value = null;
        } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
          value = parameterObject;
        } else {
          MetaObject metaObject = configuration.newMetaObject(parameterObject);
          value = metaObject.getValue(propertyName);
        }
        cacheKey.update(value);
      }
    }
    //从configuration对象中（也就是载入配置文件后存放的对象）把EnvironmentId存入
        /**
     *     <environments default="development">
     *         <environment id="development"> //就是这个id
     *             <!--当前事务交由JDBC进行管理-->
     *             <transactionManager type="JDBC"></transactionManager>
     *             <!--当前使用mybatis提供的连接池-->
     *             <dataSource type="POOLED">
     *                 <property name="driver" value="${jdbc.driver}"/>
     *                 <property name="url" value="${jdbc.url}"/>
     *                 <property name="username" value="${jdbc.username}"/>
     *                 <property name="password" value="${jdbc.password}"/>
     *             </dataSource>
     *         </environment>
     *     </environments>
     */
    if (configuration.getEnvironment() != null) {
      // issue #176
      cacheKey.update(configuration.getEnvironment().getId());
    }
    //返回
    return cacheKey;
  }
```

我们再点进去`cacheKey.update()`方法看一看

```java

public class CacheKey implements Cloneable, Serializable {
  private static final long serialVersionUID = 1146682552656046210L;
  public static final CacheKey NULL_CACHE_KEY = new NullCacheKey();
  private static final int DEFAULT_MULTIPLYER = 37;
  private static final int DEFAULT_HASHCODE = 17;

  private final int multiplier;
  private int hashcode;
  private long checksum;
  private int count;
  //值存入的地方
  private transient List<Object> updateList;
  //省略部分方法......
  //省略部分方法......
  public void update(Object object) {
    int baseHashCode = object == null ? 1 : ArrayUtil.hashCode(object); 
    count++;
    checksum += baseHashCode;
    baseHashCode *= count;
    hashcode = multiplier * hashcode + baseHashCode;
    //看到把值传入到了一个list中
    updateList.add(object);
  }
 
  //省略部分方法......
}
```

我们知道了那些数据是在CacheKey对象中如何存储的了。下面我们返回`createCacheKey()`方法。

<img src=".\img\image-20210810175452868.png" alt="image-20210810175452868" style="zoom:80%;" />



我们进入`BaseExecutor`，可以看到一个`query()`方法：

![image-20210810175532242](.\img\image-20210810175532242.png)

这里我们很清楚的看到，在执行`query()`方法前，`CacheKey`方法被创建了



我们可以看到，创建CacheKey后调用了query()方法，我们再次点进去：

![image-20210810180306207](.\img\image-20210810180306207.png)



在执行SQL前如何在一级缓存中找不到Key，那么将会执行sql，我们来看一下执行sql前后会做些什么，进入`list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);`

![image-20210810180328744](.\img\image-20210810180328744.png)

分析一下：

```java
 private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    List<E> list;
    //1. 把key存入缓存，value放一个占位符
	localCache.putObject(key, EXECUTION_PLACEHOLDER);
    try {
      //2. 与数据库交互
      list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
      //3. 如果第2步出了什么异常，把第1步存入的key删除
      localCache.removeObject(key);
    }
      //4. 把结果存入缓存
    localCache.putObject(key, list);
    if (ms.getStatementType() == StatementType.CALLABLE) {
      localOutputParameterCache.putObject(key, parameter);
    }
    return list;
  }
```

#### 一级缓存源码分析结论：

1. 一级缓存的数据结构是一个`HashMap<Object,Object>`，它的value就是查询结果，它的key是`CacheKey`，`CacheKey`中有一个**list**属性，`statementId,params,rowbounds,sql`等参数都存入到了这个**list**中
3. 先创建`CacheKey`，会首先根据`CacheKey`查询缓存中有没有，如果有，就处理缓存中的参数，如果没有，就执行sql，执行sql后执行sql后把结果存入缓存



### 二级缓存

注意：Mybatis的二级缓存不是默认开启的，是需要经过配置才能使用的

#### 启用二级缓存

分为三步走：

1）开启映射器配置文件中的缓存配置：

```xml
 <settings>
    <setting name="cacheEnabled" value="true"/>
 </settings>
```

2)  在需要使用二级缓存的Mapper配置文件中配置<cache>标签

```xml
  <!--type：cache使用的类型，默认是PerpetualCache，这在一级缓存中提到过。
      eviction： 定义回收的策略，常见的有FIFO，LRU。
      flushInterval： 配置一定时间自动刷新缓存，单位是毫秒。
      size： 最多缓存对象的个数。
      readOnly： 是否只读，若配置可读写，则需要对应的实体类能够序列化。
      blocking： 若缓存中找不到对应的key，是否会一直blocking，直到有对应的数据进入缓存。
      -->  
<cache></cache>
```

3）在具体CURD标签上配置 **useCache=true**

```xml
   <select id="findById" resultType="com.itheima.pojo.User" useCache="true">
       select * from user where id = #{id}
   </select>
```

** 注意：实体类要实现Serializable接口，因为二级缓存会将对象写进硬盘，就必须序列化，以及兼容对象在网络中的传输 



具体实现

```java
  /**
   * 测试一级缓存
   */
  @Test
  public void secondLevelCacheTest() throws IOException {

    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");

    // 2. (1)解析了配置文件，封装configuration对象 （2）创建了DefaultSqlSessionFactory工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);

    // 3.问题：openSession()执行逻辑是什么？
    // 3. (1)创建事务对象 （2）创建了执行器对象cachingExecutor (3)创建了DefaultSqlSession对象
    SqlSession sqlSession1 = sqlSessionFactory.openSession();

    // 发起第一次查询，查询ID为1的用户
    User user1 = sqlSession1.selectOne("com.itheima.mapper.UserMapper.findByCondition", 1);

    // ***必须调用sqlSession1.commit()或者close(),一级缓存中的内容才会刷新到二级缓存中
    sqlSession1.commit();// close();
    // 发起第二次查询，查询ID为1的用户
    SqlSession sqlSession2 = sqlSessionFactory.openSession();
    User user2 = sqlSession2.selectOne("com.itheima.mapper.UserMapper.findByCondition", 1);

    System.out.println(user1 == user2);
    System.out.println(user1);
    System.out.println(user2);

    sqlSession1.close();
    sqlSession2.close();


  }
```



<img src=".\img\image-20210906173921273.png" alt="image-20210906173921273" style="zoom:67%;" />



#### 二级缓存源码分析

问题：

① cache标签如何被解析的（二级缓存的底层数据结构是什么？）？

② 同时开启一级缓存二级缓存，优先级？

③ 为什么只有执行sqlSession.commit或者sqlSession.close二级缓存才会生效

④ 更新方法为什么不会清空二级缓存？

#### 标签 < cache/> 的解析

二级缓存和具体的命名空间绑定，一个Mapper中有一个Cache, 相同Mapper中的MappedStatement共用同一个Cache

根据之前的mybatis源码剖析，xml的解析工作主要交给XMLConfigBuilder.parse()方法来实现

```java
  // XMLConfigBuilder.parse()
  public Configuration parse() {
      if (parsed) {
          throw new BuilderException("Each XMLConfigBuilder can only be used once.");
      }
      parsed = true;
      parseConfiguration(parser.evalNode("/configuration"));// 在这里
      return configuration;
  }
  
 // parseConfiguration()
 // 既然是在xml中添加的，那么我们就直接看关于mappers标签的解析
 private void parseConfiguration(XNode root) {
     try {
         Properties settings = settingsAsPropertiess(root.evalNode("settings"));
         propertiesElement(root.evalNode("properties"));
         loadCustomVfs(settings);
         typeAliasesElement(root.evalNode("typeAliases"));
         pluginElement(root.evalNode("plugins"));
         objectFactoryElement(root.evalNode("objectFactory"));
         objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
         reflectionFactoryElement(root.evalNode("reflectionFactory"));
         settingsElement(settings);
         // read it after objectFactory and objectWrapperFactory issue #631
         environmentsElement(root.evalNode("environments"));
         databaseIdProviderElement(root.evalNode("databaseIdProvider"));
         typeHandlerElement(root.evalNode("typeHandlers"));
         // 就是这里
         mapperElement(root.evalNode("mappers"));
     } catch (Exception e) {
         throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
     }
 }
 
 
 // mapperElement()
 private void mapperElement(XNode parent) throws Exception {
     if (parent != null) {
         for (XNode child : parent.getChildren()) {
             if ("package".equals(child.getName())) {
                 String mapperPackage = child.getStringAttribute("name");
                 configuration.addMappers(mapperPackage);
             } else {
                 String resource = child.getStringAttribute("resource");
                 String url = child.getStringAttribute("url");
                 String mapperClass = child.getStringAttribute("class");
                 // 按照我们本例的配置，则直接走该if判断
                 if (resource != null && url == null && mapperClass == null) {
                     ErrorContext.instance().resource(resource);
                     InputStream inputStream = Resources.getResourceAsStream(resource);
                     XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
                     // 生成XMLMapperBuilder，并执行其parse方法
                     mapperParser.parse();
                 } else if (resource == null && url != null && mapperClass == null) {
                     ErrorContext.instance().resource(url);
                     InputStream inputStream = Resources.getUrlAsStream(url);
                     XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
                     mapperParser.parse();
                 } else if (resource == null && url == null && mapperClass != null) {
                     Class<?> mapperInterface = Resources.classForName(mapperClass);
                     configuration.addMapper(mapperInterface);
                 } else {
                     throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
                 }
             }
         }
     }
 } 
```

我们来看看解析Mapper.xml

```java
// XMLMapperBuilder.parse()
public void parse() {
    if (!configuration.isResourceLoaded(resource)) {
        // 解析mapper属性
        configurationElement(parser.evalNode("/mapper"));
        configuration.addLoadedResource(resource);
        bindMapperForNamespace();
    }
 
    parsePendingResultMaps();
    parsePendingChacheRefs();
    parsePendingStatements();
}
 
// configurationElement()
private void configurationElement(XNode context) {
    try {
        String namespace = context.getStringAttribute("namespace");
        if (namespace == null || namespace.equals("")) {
            throw new BuilderException("Mapper's namespace cannot be empty");
        }
        builderAssistant.setCurrentNamespace(namespace);
        cacheRefElement(context.evalNode("cache-ref"));
        // 最终在这里看到了关于cache属性的处理
        cacheElement(context.evalNode("cache"));
        parameterMapElement(context.evalNodes("/mapper/parameterMap"));
        resultMapElements(context.evalNodes("/mapper/resultMap"));
        sqlElement(context.evalNodes("/mapper/sql"));
        // 这里会将生成的Cache包装到对应的MappedStatement
        buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing Mapper XML. Cause: " + e, e);
    }
}
 
// cacheElement()
private void cacheElement(XNode context) throws Exception {
    if (context != null) {
        //解析<cache/>标签的type属性，这里我们可以自定义cache的实现类，比如redisCache，如果没有自定义，这里使用和一级缓存相同的PERPETUAL
        String type = context.getStringAttribute("type", "PERPETUAL");
        Class<? extends Cache> typeClass = typeAliasRegistry.resolveAlias(type);
        String eviction = context.getStringAttribute("eviction", "LRU");
        Class<? extends Cache> evictionClass = typeAliasRegistry.resolveAlias(eviction);
        Long flushInterval = context.getLongAttribute("flushInterval");
        Integer size = context.getIntAttribute("size");
        boolean readWrite = !context.getBooleanAttribute("readOnly", false);
        boolean blocking = context.getBooleanAttribute("blocking", false);
        Properties props = context.getChildrenAsProperties();
        // 构建Cache对象
        builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, blocking, props);
    }
}
```

先来看看是如何构建Cache对象的

**MapperBuilderAssistant.useNewCache()**

```java
public Cache useNewCache(Class<? extends Cache> typeClass,
                         Class<? extends Cache> evictionClass,
                         Long flushInterval,
                         Integer size,
                         boolean readWrite,
                         boolean blocking,
                         Properties props) {
    // 1.生成Cache对象
    Cache cache = new CacheBuilder(currentNamespace)
         //这里如果我们定义了<cache/>中的type，就使用自定义的Cache,否则使用和一级缓存相同的PerpetualCache
        .implementation(valueOrDefault(typeClass, PerpetualCache.class))
        .addDecorator(valueOrDefault(evictionClass, LruCache.class))
        .clearInterval(flushInterval)
        .size(size)
        .readWrite(readWrite)
        .blocking(blocking)
        .properties(props)
        .build();
    // 2.添加到Configuration中
    configuration.addCache(cache);
    // 3.并将cache赋值给MapperBuilderAssistant.currentCache
    currentCache = cache;
    return cache;
}
```

我们看到一个Mapper.xml只会解析一次<cache/>标签，也就是只创建一次Cache对象，放进configuration中，并将cache赋值给MapperBuilderAssistant.currentCache

##### buildStatementFromContext(context.evalNodes("select|insert|update|delete"));将Cache包装到MappedStatement

```java
// buildStatementFromContext()
private void buildStatementFromContext(List<XNode> list) {
    if (configuration.getDatabaseId() != null) {
        buildStatementFromContext(list, configuration.getDatabaseId());
    }
    buildStatementFromContext(list, null);
}
 
//buildStatementFromContext()
private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
    for (XNode context : list) {
        final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
        try {
            // 每一条执行语句转换成一个MappedStatement
            statementParser.parseStatementNode();
        } catch (IncompleteElementException e) {
            configuration.addIncompleteStatement(statementParser);
        }
    }
}
 
// XMLStatementBuilder.parseStatementNode();
public void parseStatementNode() {
    String id = context.getStringAttribute("id");
    String databaseId = context.getStringAttribute("databaseId");
    ...
 
    Integer fetchSize = context.getIntAttribute("fetchSize");
    Integer timeout = context.getIntAttribute("timeout");
    String parameterMap = context.getStringAttribute("parameterMap");
    String parameterType = context.getStringAttribute("parameterType");
    Class<?> parameterTypeClass = resolveClass(parameterType);
    String resultMap = context.getStringAttribute("resultMap");
    String resultType = context.getStringAttribute("resultType");
    String lang = context.getStringAttribute("lang");
    LanguageDriver langDriver = getLanguageDriver(lang);
 
    ...
    // 创建MappedStatement对象
    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
                                        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
                                        resultSetTypeEnum, flushCache, useCache, resultOrdered, 
                                        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
}
 
// builderAssistant.addMappedStatement()
public MappedStatement addMappedStatement(
    String id,
    ...) {
 
    if (unresolvedCacheRef) {
        throw new IncompleteElementException("Cache-ref not yet resolved");
    }
 
    id = applyCurrentNamespace(id, false);
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
    //创建MappedStatement对象
    MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration, id, sqlSource, sqlCommandType)
        ...
        .flushCacheRequired(valueOrDefault(flushCache, !isSelect))
        .useCache(valueOrDefault(useCache, isSelect))
        .cache(currentCache);// 在这里将之前生成的Cache封装到MappedStatement
 
    ParameterMap statementParameterMap = getStatementParameterMap(parameterMap, parameterType, id);
    if (statementParameterMap != null) {
        statementBuilder.parameterMap(statementParameterMap);
    }
 
    MappedStatement statement = statementBuilder.build();
    configuration.addMappedStatement(statement);
    return statement;
}
```

我们看到将Mapper中创建的Cache对象，加入到了每个MappedStatement对象中，也就是同一个Mapper中所有的MappedStatement中的cache属性引用的是同一个

有关于<cache/>标签的解析就到这了。



#### 查询源码分析

##### **CachingExecutor**

```java
// CachingExecutor
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    BoundSql boundSql = ms.getBoundSql(parameterObject);
    // 创建 CacheKey
    CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
    return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}

public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
    throws SQLException {
    // 从 MappedStatement 中获取 Cache，注意这里的 Cache 是从MappedStatement中获取的
    // 也就是我们上面解析Mapper中<cache/>标签中创建的，它保存在Configration中
    // 我们在上面解析blog.xml时分析过每一个MappedStatement都有一个Cache对象，就是这里
    Cache cache = ms.getCache();
    // 如果配置文件中没有配置 <cache>，则 cache 为空
    if (cache != null) {
        //如果需要刷新缓存的话就刷新：flushCache="true"
        flushCacheIfRequired(ms);
        if (ms.isUseCache() && resultHandler == null) {
            ensureNoOutParams(ms, boundSql);
            // 访问二级缓存
            List<E> list = (List<E>) tcm.getObject(cache, key);
            // 缓存未命中
            if (list == null) {
                // 如果没有值，则执行查询，这个查询实际也是先走一级缓存查询，一级缓存也没有的话，则进行DB查询
                list = delegate.<E>query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                // 缓存查询结果
                tcm.putObject(cache, key, list);
            }
            return list;
        }
    }
    return delegate.<E>query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

如果设置了**flushCache="true"，则每次查询都会刷新缓存**

```xml
<!-- 执行此语句清空缓存 -->
<select id="findbyId" resultType="com.itheima.pojo.user" useCache="true" flushCache="true" >
    select * from t_demo
</select>
```

如上，注意二级缓存是从 MappedStatement 中获取的。由于 MappedStatement 存在于全局配置中，可以多个 CachingExecutor 获取到，这样就会出现线程安全问题。除此之外，若不加以控制，多个事务共用一个缓存实例，会导致脏读问题。至于脏读问题，需要借助其他类来处理，也就是上面代码中 tcm 变量对应的类型。下面分析一下。

##### **TransactionalCacheManager**

```java
/** 事务缓存管理器 */
public class TransactionalCacheManager {

    // Cache 与 TransactionalCache 的映射关系表
    private final Map<Cache, TransactionalCache> transactionalCaches = new HashMap<Cache, TransactionalCache>();

    public void clear(Cache cache) {
        // 获取 TransactionalCache 对象，并调用该对象的 clear 方法，下同
        getTransactionalCache(cache).clear();
    }

    public Object getObject(Cache cache, CacheKey key) {
        // 直接从TransactionalCache中获取缓存
        return getTransactionalCache(cache).getObject(key);
    }

    public void putObject(Cache cache, CacheKey key, Object value) {
        // 直接存入TransactionalCache的缓存中
        getTransactionalCache(cache).putObject(key, value);
    }

    public void commit() {
        for (TransactionalCache txCache : transactionalCaches.values()) {
            txCache.commit();
        }
    }

    public void rollback() {
        for (TransactionalCache txCache : transactionalCaches.values()) {
            txCache.rollback();
        }
    }

    private TransactionalCache getTransactionalCache(Cache cache) {
        // 从映射表中获取 TransactionalCache
        TransactionalCache txCache = transactionalCaches.get(cache);
        if (txCache == null) {
            // TransactionalCache 也是一种装饰类，为 Cache 增加事务功能
            // 创建一个新的TransactionalCache，并将真正的Cache对象存进去
            txCache = new TransactionalCache(cache);
            transactionalCaches.put(cache, txCache);
        }
        return txCache;
    }
}
```

TransactionalCacheManager 内部维护了 Cache 实例与 TransactionalCache 实例间的映射关系，该类也仅负责维护两者的映射关系，真正做事的还是 TransactionalCache。TransactionalCache 是一种缓存装饰器，可以为 Cache 实例增加事务功能。下面分析一下该类的逻辑。

##### **TransactionalCache**

```java
public class TransactionalCache implements Cache {
    //真正的缓存对象，和上面的Map<Cache, TransactionalCache>中的Cache是同一个
    private final Cache delegate;
    private boolean clearOnCommit;
    // 在事务被提交前，所有从数据库中查询的结果将缓存在此集合中
    private final Map<Object, Object> entriesToAddOnCommit;
    // 在事务被提交前，当缓存未命中时，CacheKey 将会被存储在此集合中
    private final Set<Object> entriesMissedInCache;


    @Override
    public Object getObject(Object key) {
        // 查询的时候是直接从delegate中去查询的，也就是从真正的缓存对象中查询
        Object object = delegate.getObject(key);
        if (object == null) {
            // 缓存未命中，则将 key 存入到 entriesMissedInCache 中
            entriesMissedInCache.add(key);
        }

        if (clearOnCommit) {
            return null;
        } else {
            return object;
        }
    }

    @Override
    public void putObject(Object key, Object object) {
        // 将键值对存入到 entriesToAddOnCommit 这个Map中中，而非真实的缓存对象 delegate 中
        entriesToAddOnCommit.put(key, object);
    }

    @Override
    public Object removeObject(Object key) {
        return null;
    }

    @Override
    public void clear() {
        clearOnCommit = true;
        // 清空 entriesToAddOnCommit，但不清空 delegate 缓存
        entriesToAddOnCommit.clear();
    }

    public void commit() {
        // 根据 clearOnCommit 的值决定是否清空 delegate
        if (clearOnCommit) {
            delegate.clear();
        }
        
        // 刷新未缓存的结果到 delegate 缓存中
        flushPendingEntries();
        // 重置 entriesToAddOnCommit 和 entriesMissedInCache
        reset();
    }

    public void rollback() {
        unlockMissedEntries();
        reset();
    }

    private void reset() {
        clearOnCommit = false;
        // 清空集合
        entriesToAddOnCommit.clear();
        entriesMissedInCache.clear();
    }

    private void flushPendingEntries() {
        for (Map.Entry<Object, Object> entry : entriesToAddOnCommit.entrySet()) {
            // 将 entriesToAddOnCommit 中的内容转存到 delegate 中
            delegate.putObject(entry.getKey(), entry.getValue());
        }
        for (Object entry : entriesMissedInCache) {
            if (!entriesToAddOnCommit.containsKey(entry)) {
                // 存入空值
                delegate.putObject(entry, null);
            }
        }
    }

    private void unlockMissedEntries() {
        for (Object entry : entriesMissedInCache) {
            try {
                // 调用 removeObject 进行解锁
                delegate.removeObject(entry);
            } catch (Exception e) {
                log.warn("...");
            }
        }
    }

}
```

存储二级缓存对象的时候是放到了TransactionalCache.entriesToAddOnCommit这个map中，但是每次查询的时候是直接从TransactionalCache.delegate中去查询的，所以这个二级缓存查询数据库后，设置缓存值是没有立刻生效的，主要是因为直接存到 delegate 会导致脏数据问题



#### 为何只有SqlSession提交或关闭之后？

那我们来看下SqlSession.commit()方法做了什么

**SqlSession**

```java
@Override
public void commit(boolean force) {
    try {
        // 主要是这句
        executor.commit(isCommitOrRollbackRequired(force));
        dirty = false;
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error committing transaction.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
 
// CachingExecutor.commit()
@Override
public void commit(boolean required) throws SQLException {
    delegate.commit(required);
    tcm.commit();// 在这里
}
 
// TransactionalCacheManager.commit()
public void commit() {
    for (TransactionalCache txCache : transactionalCaches.values()) {
        txCache.commit();// 在这里
    }
}
 
// TransactionalCache.commit()
public void commit() {
    if (clearOnCommit) {
        delegate.clear();
    }
    flushPendingEntries();//这一句
    reset();
}
 
// TransactionalCache.flushPendingEntries()
private void flushPendingEntries() {
    for (Map.Entry<Object, Object> entry : entriesToAddOnCommit.entrySet()) {
        // 在这里真正的将entriesToAddOnCommit的对象逐个添加到delegate中，只有这时，二级缓存才真正的生效
        delegate.putObject(entry.getKey(), entry.getValue());
    }
    for (Object entry : entriesMissedInCache) {
        if (!entriesToAddOnCommit.containsKey(entry)) {
            delegate.putObject(entry, null);
        }
    }
}
```

#### 二级缓存的刷新

我们来看看SqlSession的更新操作

```java
public int update(String statement, Object parameter) {
    int var4;
    try {
        this.dirty = true;
        MappedStatement ms = this.configuration.getMappedStatement(statement);
        var4 = this.executor.update(ms, this.wrapCollection(parameter));
    } catch (Exception var8) {
        throw ExceptionFactory.wrapException("Error updating database.  Cause: " + var8, var8);
    } finally {
        ErrorContext.instance().reset();
    }

    return var4;
}

public int update(MappedStatement ms, Object parameterObject) throws SQLException {
    this.flushCacheIfRequired(ms);
    return this.delegate.update(ms, parameterObject);
}

private void flushCacheIfRequired(MappedStatement ms) {
    //获取MappedStatement对应的Cache，进行清空
    Cache cache = ms.getCache();
    //SQL需设置flushCache="true" 才会执行清空
    if (cache != null && ms.isFlushCacheRequired()) {
  this.tcm.clear(cache);
    }
}
```

MyBatis二级缓存只适用于不常进行增、删、改的数据，比如国家行政区省市区街道数据。一但数据变更，MyBatis会清空缓存。因此二级缓存不适用于经常进行更新的数据。

#### 总结：

在二级缓存的设计上，MyBatis大量地运用了装饰者模式，如CachingExecutor, 以及各种Cache接口的装饰器。

- 二级缓存实现了Sqlsession之间的缓存数据共享，属于namespace级别
- 二级缓存具有丰富的缓存策略。
- 二级缓存可由多个装饰器，与基础缓存组合而成
- 二级缓存工作由 一个缓存装饰执行器CachingExecutor和 一个事务型预缓存TransactionalCache 完成























