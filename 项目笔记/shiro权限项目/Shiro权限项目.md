# Shiro权限项目

## 环境配置

### spring容器

先在`resources`文件夹新建`spring.xml`文件，内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

  <context:property-placeholder location="classpath:jdbc.properties"/>
  <context:property-placeholder location="classpath:shiro-config.properties"/>

  <import resource="spring-mybatis.xml"/>
  <import resource="spring-shiro.xml"/>

</beans>
```

==注意==：property-placeholder的location值要加上`classpath:`，否则可能会出现FileNotFoundException: Could not open ServletContext resource [/jdbc.properties]异常

上面其beans里的内容在暂时先可以无视，接着配置web.xml，添加上下文监听器

web.xml

```xml
<!--spring-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring.xml</param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```

顺带可以统一一下编码格式

```xml
<!--编码过滤器-->
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <description>字符集编码</description>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### springmvc

新建一个`spring-mvc.xml`文件，内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

  <mvc:annotation-driven/>

  <!--静态资源-->
  <mvc:default-servlet-handler/>

  <context:component-scan base-package="com.hemou.**.controller"/>

  <!--返回数据转为json格式-->
  <mvc:annotation-driven>
    <mvc:message-converters>
      <bean class="org.springframework.http.converter.StringHttpMessageConverter"/>
      <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
    </mvc:message-converters>
  </mvc:annotation-driven>

  <!--视图解释器 -->
  <bean id="freeMarkerViewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <property name="viewClass" value="com.hemou.core.freemarker.FreemarkerViewExtend"/>
    <property name="contentType" value="text/html;charset=UTF-8"/>
    <property name="cache" value="true"/>
    <property name="suffix" value=".ftl"/>
    <property name="order" value="0"/>
  </bean>

  <!--freeMarker配置-->
  <bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPaths" value="/WEB-INF/ftl/"/>
    <property name="defaultEncoding" value="utf-8"/>
  </bean>

  <!--文件上传-->
  <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"/>

</beans>
```

如果之后运行项目发现任然出现`No converter found for return value of type`这样的错误，则检查@ResponseBody方法返回的对象是否写了Getter和Setter方法。

接着配置web.xml，新增内容

```xml
<!-- spring mvc servlet -->
<servlet>
    <servlet-name>springMvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springMvc</servlet-name>
    <url-pattern>*.shtml</url-pattern>
</servlet-mapping>
```

### freemarker

**配置全局变量**

上面配置的freemarker视图解析器

spring-mvc.xml 部分

```xml
<bean id="freeMarkerViewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <property name="viewClass" value="com.hemou.core.freemarker.FreemarkerViewExtend"/>
    ...
  </bean>
```

需要建一个继承`FreeMarkerView`的类，并重写`exposeHelpers`这样就可以把自己想要的数据放到model，方便渲染。如下配置就可以把ftl文件中的`${basePath}`替换为项目路径。

FreemarkerViewExtend.java

```java
public class FreemarkerViewExtend extends FreeMarkerView {
    @Override
    protected void exposeHelpers(Map<String, Object> model, HttpServletRequest request) throws Exception {
        super.exposeHelpers(model, request);
        if(TokenManager.isLogin())model.put("token", TokenManager.getToken());
        model.put("basePath", request.getContextPath());
    }    
}
```

**自动装载**

一般我们通过macro自定义的指令需要使用import进行导入，但是懒得的写的import的话可以使用自动装载

spring-mvc.xml 部分

```xml
<!--freeMarker配置-->
  <bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPaths" value="/WEB-INF/ftl/"/>
    <property name="defaultEncoding" value="utf-8"/>
    <property name="freemarkerSettings">
      <props>
        <!-- 自动装载，引入Freemarker，用于Freemarker Macro引入 -->
        <prop key="auto_import">
          /common/config/top.ftl as _top,
          /common/config/left.ftl as _left
        </prop>
      </props>
    </property>
  </bean>
```

这样我们就可以用`_top`直接引用`/common/config/top.ftl`中所定义的指令，而无需import

/common/config/top.ftl

```F#
<#macro top index >
	...
</#macro>
```

引用处

```F#
<@_top.top 1/>
```

**shiro标签**

要想结合freemarker使用shiro相关的标签则必须修改上面id为freemarkerConfig 的bean，使其class属性为继承了FreeMarkerConfigurer的类

首先以来肯定是少不了的

pom.xml

```xml
<!-- freemarker + shiro（标签） begin -->
<dependency>
    <groupId>net.mingsoft</groupId>
    <artifactId>shiro-freemarker-tags</artifactId>
    <version>0.1</version>
</dependency>
```

spring-mvc.xml 部分

```xml
<!--freeMarker配置-->
<bean id="freemarkerConfig" class="com.hemou.core.freemarker.FreeMarkerConfigurerExtend">
  <property name="templateLoaderPaths" value="/WEB-INF/ftl/"/>
  <property name="defaultEncoding" value="utf-8"/>
  <property name="freemarkerSettings">
    <props>
      <!-- 自动装载，引入Freemarker，用于Freemarker Macro引入 -->
      <prop key="auto_import">
        /common/config/top.ftl as _top,
        /common/config/left.ftl as _left
      </prop>
    </props>
  </property>
</bean>
```

FreeMarkerConfigurerExtend.java

```java
public class FreeMarkerConfigurerExtend extends FreeMarkerConfigurer {
    @Override
    public void afterPropertiesSet() throws IOException, TemplateException {
        super.afterPropertiesSet();
        Configuration cfg = this.getConfiguration();
        // 添加shiro标签
        cfg.setSharedVariable("shiro", new ShiroTags());
    }
}
```

这样我们就可以在ftl中写关于shiro的标签了

### mybatis

新建一个`spring-mybatis.xml`文件，内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

  <context:component-scan base-package="com.hemou.**.service"/>

  <!-- 配置数据源 -->
  <bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
    <!--		<property name="driverClassName" value="${jdbc.driverClassName}" /> -->
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <property name="initialSize" value="${jdbc.initialSize}"/>
    <property name="minIdle" value="${jdbc.minIdle}"/>
    <property name="maxActive" value="${jdbc.maxActive}"/>
    <property name="maxWait" value="${jdbc.maxWait}"/>
    <property name="timeBetweenEvictionRunsMillis" value="${jdbc.timeBetweenEvictionRunsMillis}"/>
    <property name="minEvictableIdleTimeMillis" value="${jdbc.minEvictableIdleTimeMillis}"/>
    <property name="validationQuery" value="${jdbc.validationQuery}"/>
    <property name="testWhileIdle" value="${jdbc.testWhileIdle}"/>
    <property name="testOnBorrow" value="${jdbc.testOnBorrow}"/>
    <property name="testOnReturn" value="${jdbc.testOnReturn}"/>
    <property name="removeAbandoned" value="${jdbc.removeAbandoned}"/>
    <property name="removeAbandonedTimeout" value="${jdbc.removeAbandonedTimeout}"/>
    <!--	    <property name="logAbandoned" value="${jdbc.logAbandoned}" /> -->
    <property name="filters" value="${jdbc.filters}"/>
    <!-- 关闭abanded连接时输出错误日志 -->
    <property name="logAbandoned" value="true"/>
    <property name="proxyFilters">
      <list>
        <ref bean="log-filter"/>
      </list>
    </property>
    <!-- 监控数据库 -->
    <!-- <property name="filters" value="stat" /> -->
    <!--		<property name="filters" value="mergeStat" />-->
  </bean>
  <bean id="log-filter" class="com.alibaba.druid.filter.logging.Log4jFilter">
    <property name="resultSetLogEnabled" value="true" />
  </bean>

  <!--sqlSessionFactory-->
  <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="typeAliasesPackage" value="com.hemou.common.model"/>
    <property name="dataSource" ref="dataSource"/>
    <property name="mapperLocations" value="classpath:com/hemou/common/mapper/*.xml"/>
  </bean>

  <!--Mybatis的mapper扫描-->
  <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.hemou.common.dao"/>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryBean"/>
  </bean>

  <!-- 启动spring的事务管理   配置都是固定的在哪个项目都几乎一样 -->
  <aop:config>
    <aop:pointcut id="serviceMethod" expression="execution(* com.hemou..service.impl.*.*(..))" />
    <aop:advisor advice-ref="adviceTran" pointcut-ref="serviceMethod" />
  </aop:config>

  <!-- 定义事务管理器 -->
  <bean id="trans" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
  </bean>

  <!--配置advice-->
  <tx:advice id="adviceTran" transaction-manager="trans">
    <tx:attributes>
      <tx:method name="get*" propagation="SUPPORTS" isolation="DEFAULT"/>
      <tx:method name="select*" propagation="SUPPORTS" isolation="DEFAULT"/>
      <tx:method name="update*" propagation="REQUIRED"/>
      <tx:method name="del*" propagation="REQUIRED"/>
      <tx:method name="insert*" propagation="REQUIRED"/>
      <tx:method name="*" propagation="SUPPORTS" read-only="true"/>
    </tx:attributes>
  </tx:advice>

</beans>
```

### shiro

新建一个`spring-shiro.xml`文件，内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd">

  <!-- 授权 认证 -->
  <bean id="simpleRealm" class="com.hemou.core.shiro.token.SimpleRealm">
    <property name="credentialsMatcher">
      <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
        <!--加密算法-->
        <property name="hashAlgorithmName" value="MD5"/>
        <!--加密次数-->
        <property name="hashIterations" value="1024"/>
      </bean>
    </property>
  </bean>

  <!-- 用户信息记住我功能的相关配置 -->
  <bean id="rememberMeCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
    <constructor-arg value="v_v-re-baidu"/>
    <property name="httpOnly" value="true"/>
    <!-- 配置存储rememberMe Cookie的domain为 一级域名
    <property name="domain" value=".itboy.net"/>
     -->
    <property name="maxAge" value="2592000"/><!-- 30天时间，记住我30天 -->
  </bean>

  <!-- rememberMe管理器 -->
  <bean id="rememberMeManager" class="org.apache.shiro.web.mgt.CookieRememberMeManager">
    <!-- rememberMe cookie加密的密钥 建议每个项目都不一样 默认AES算法 密钥长度（128 256 512 位）-->
    <!--<property name="cipherKey"-->
    <!--          value="#{T(org.apache.shiro.codec.Base64).decode('3AvVhmFLUs0KTA3Kprsdag==')}"/>-->
    <property name="cookie" ref="rememberMeCookie"/>
  </bean>

  <!-- 安全管理器 -->
  <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="realm" ref="simpleRealm"/>
    <!--<property name="sessionManager" ref="sessionManager"/>-->
    <property name="rememberMeManager" ref="rememberMeManager"/>
    <property name="cacheManager" ref="cacheManager"/>
  </bean>

  <bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
    <!-- Set a net.sf.ehcache.CacheManager instance here if you already have one.  If not, a new one
             will be creaed with a default config:
             <property name="cacheManager" ref="ehCacheManager"/> -->
    <!-- If you don't have a pre-built net.sf.ehcache.CacheManager instance to inject, but you want
             a specific Ehcache configuration to be used, specify that here.  If you don't, a default
             will be used.: -->
    <property name="cacheManagerConfigFile" value="classpath:ehcache.xml"/>
  </bean>


  <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="securityManager" ref="securityManager" />
    <property name="loginUrl" value="/u/login.shtml" />
    <property name="successUrl" value="/" />
    <property name="unauthorizedUrl" value="/?login" />

    <!--	初始配置，现采用自定义	-->
    <property name="filterChainDefinitions" >
      <value>
        /** = anon

        <!--/page/login.jsp = anon-->
        <!--/page/register/* = anon-->
        <!--/page/index.jsp = authc-->
        <!--/page/addItem* = authc,roles[数据管理员]-->
        <!--/page/file* = authc,roleOR[普通用户,数据管理员]-->
        <!--/page/listItems* = authc,roleOR[数据管理员,普通用户]-->
        <!--/page/showItem* = authc,roleOR[数据管理员,普通用户]-->
        <!--/page/updateItem*=authc,roles[数据管理员]-->
      </value>
    </property>
    <!-- 读取初始自定义权限内容-->
    <!--<property name="filterChainDefinitions" value="#{shiroManager.loadFilterChainDefinitions()}"/>-->
    <!--<property name="filters">-->
    <!--  <util:map>-->
    <!--    <entry key="login" value-ref="login"></entry>-->
    <!--    <entry key="role" value-ref="role"></entry>-->
    <!--    <entry key="simple" value-ref="simple"></entry>-->
    <!--    <entry key="permission" value-ref="permission"></entry>-->
    <!--    <entry key="kickout" value-ref="kickoutSessionFilter"></entry>-->
    <!--  </util:map>-->
    <!--</property>-->
  </bean>
</beans>
```

然后 配置web.xml，新添加一个fitler

web.xml

```xml
<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

需要注意的是，==上面spring-shiro.xml中配置的ShiroFilterFactoryBean的bean的id名必须与filter-name一样，不然会报错==

还有就是配置rememberMeCookie时，一定要填写构造参数，也就是constructor-arg的值

spring-shiro.xml 部分

```xml
<!-- 用户信息记住我功能的相关配置 -->
  <bean id="rememberMeCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
    <constructor-arg value="v_v-re-baidu"/>
    <property name="httpOnly" value="true"/>
    <!-- 配置存储rememberMe Cookie的domain为 一级域名
    <property name="domain" value=".itboy.net"/>
     -->
    <property name="maxAge" value="2592000"/><!-- 30天时间，记住我30天 -->
  </bean>
```

不然会报类似如下错误，排查了好久才发现:sweat:

[WARN]\[2021-01-08 15:57:43,346][org.apache.shiro.mgt.DefaultSecurityManager]Delegate RememberMeManager instance of type [org.apache.shiro.web.mgt.CookieRememberMeManager] threw an exception during onSuccessfulLogin.  RememberMe services will not be performed for account  .... java.lang.IllegalStateException: Cookie name cannot be null/empty.

## 工具类

### TokenManager.java

```java
public class TokenManager {

    public static Subject getSubject(){
        return SecurityUtils.getSubject();
    }

    public static UUser getToken(){
        return (UUser) getSubject().getPrincipal();
    }

    /**
     * 获取Sessoin
     * @return
     */
    public static Session getSession(){
        return getSubject().getSession();
    }

    /**
     * 将数据存入Session
     * @param key
     * @param value
     */
    public static void setValue(Object key, Object value){
        getSession().setAttribute(key, value);
    }

    /**
     * 从Session获取数据
     * @param key
     * @return
     */
    public static Object getValue(Object key){
        return getSession().getAttribute(key);
    }

    /**
     * 登录
     * @param user
     * @param remember
     * @return
     */
    public static UUser login(UUser user, boolean remember){
        ShiroToken shiroToken = new ShiroToken(user.getEmail(), user.getPswd());
        shiroToken.setRememberMe(remember);
        getSubject().login(shiroToken);
        return getToken();
    }

    /**
     * 登出
     */
    public static void logout(){
        getSubject().logout();
    }
}
```

在这里定义一些常用的操作，方便后面调用

### Result.java

```java
public class Result {
    private int status;
    private Object obj;
    private String msg;
    
    public Result(int status, String msg, Object obj) {
        this.msg = msg;
        this.status = status;
        this.obj = obj;
    }
    public static Result success(String message, Object obj){
        return new Result(HTTPConstant.REQUEST_SUCCESS, message, obj);
    }
    public static Result success(String message){
        return success(message, null);
    }
    public static Result warning(String msg){
        return new Result(HTTPConstant.REQUEST_WARNING, msg, null);
    }
    public static Result warning(){
        return warning(null);
    }
    public static Result error(String msg){
        return new Result(HTTPConstant.REQUEST_ERROR, msg, null);
    }
    public static Result error(){
        return error(null);
    }
    ...省略get和set方法
```

这样我们就可以统一所有返回的数据，避免响应数据结构混乱

## 功能实现

### 登录

首先梳理一下流程

1、访问登录页面 localhost:8080/shiro/u/login.shtml

2、spring-mvc拦截以shtml结尾的请求，返回login.ftl

web.xml 部分内容

```xml
<!-- spring mvc servlet -->
<servlet>
    <servlet-name>springMvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springMvc</servlet-name>
    <url-pattern>*.shtml</url-pattern>
</servlet-mapping>
```

AccoutController.java

```java
@RequestMapping("u")
@Controller
public class AccountController extends BaseController {

    @Autowired
    private UUserService userService;

    @GetMapping("/login")
    public String toLogin(){
        return "login";
    }
    ...
```

3、在login.ftl是登录表单，填写完毕点击登录按钮后发送请求 [post]localhost:8080/shiro/u/submitLogin.shtml

4、springmvc拦截请求，并做相应处理

```java
@RequestMapping("u")
@Controller
public class AccountController extends BaseController {
    @Autowired
    private UUserService userService;
    ...
        
    @ResponseBody
    @PostMapping("submitLogin")
    public Object submitLogin(boolean rememberMe, UUser entity, HttpServletRequest request){
        try {
            UUser user = TokenManager.login(entity, rememberMe);

            // 更新最后登录时间
            user.setLastLoginTime(new Date());
            userService.updateByPrimaryKeySelective(user);

            SavedRequest savedRequest = WebUtils.getSavedRequest(request);
            String url = null;
            if(null != savedRequest){
                url = savedRequest.getRequestUrl();
            }
            LoggerUtils.info(getClass(), "submitLogin：之前的访问路径 [%s]", url);
            if(StringUtils.isEmpty(url)) url = "user/index.shtml";
            return Result.success("登录成功！", url);
        } catch (UnknownAccountException e){
            return Result.error(e.getMessage());
        } catch (IncorrectCredentialsException e) {
            return Result.error("密码错误！");
        } catch (LockedAccountException e){
            return Result.error(e.getMessage());
        }
    }
    ...
```

4.1、首先进行shiro登录，这里使用上面的TokenManage工具类

TokenManage.java 部分

```java
public static Subject getSubject(){
    return SecurityUtils.getSubject();
}

public static UUser login(UUser user, boolean remember){
    ShiroToken shiroToken = new ShiroToken(user.getEmail(), user.getPswd());
    shiroToken.setRememberMe(remember);
    getSubject().login(shiroToken);
    return getToken();
}
```

4.1.1、当调用`getSubject().login(shiroToken);`时

1). 实际上是执行继承了 org.apache.shiro.realm.AuthenticatingRealm 的类
2). 重写的 doGetAuthenticationInfo(AuthenticationToken) 方法. 

问题一：为什么要继承`AuthenticatingRealm `类，而不是别的类

答：个人理解，因为继承此类会实现 认证doGetAuthenticationInfo 和 授权doGetAuthorizationInfo 两个方法，更方便

问题二：框架如何知道去执行继承了AuthenticatingRealm 下的方法

答：通过spring-shiro.xml的配置

```xml
<!-- 授权 认证 -->
<bean id="simpleRealm" class="com.hemou.core.shiro.token.SimpleRealm">
    <property name="credentialsMatcher">
        <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
            <!--加密算法-->
            <property name="hashAlgorithmName" value="MD5"/>
            <!--加密次数-->
            <property name="hashIterations" value="1024"/>
        </bean>
    </property>
</bean>

<!-- 安全管理器 -->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="realm" ref="simpleRealm"/>
    <!--<property name="sessionManager" ref="sessionManager"/>-->
    <property name="rememberMeManager" ref="rememberMeManager"/>
    <property name="cacheManager" ref="cacheManager"/>
</bean>
```

在id为securityManager的bean中配置realm，其值为simpleRealm， 而id为simpleRealm的bean就是继承了AuthenticatingRealm 的类

4.1.2、开始认证

由4.1.1知执行了SecurityUtils.getSubject().login(token)方法后，会自动调用继承了AuthorizingRealm类的doGetAuthenticationInfo方法，此方法是用来认证，也就是登录的。

ShiroToken.java

```java
public class ShiroToken extends UsernamePasswordToken implements Serializable {

    public ShiroToken(String username, String password) {
        super(username, password);
    }

    public String getPassWord(){
        return String.valueOf(this.getPassword());
    }
}
```

SimpleRealm.java

```java
public class SimpleRealm extends AuthorizingRealm {
    @Autowired
    private UUserService userService;

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        ShiroToken token = (ShiroToken)authenticationToken;
        UUser user = userService.selectByEmail(token.getUsername());
        if(null == user){
            throw new UnknownAccountException("用户不存在！");
        }else if(UUser.INVALID.equals(user.getStatus())){
            throw new LockedAccountException("账号已禁止使用！");
        }
        return new SimpleAuthenticationInfo(user, user.getPswd(), getName());
    }
    ...
}
```

在这里主要注意的一点就是：==shiro会自动帮我们完成密码匹配，所以我们只需要传入正确的密码即可==，下面是方法具体逻辑

1）首先将authenticationToken强转为ShiroToken

2）通过ShiroToken的getUsername()方法获取唯一标识，可以查看上面的TokenManager，在这里我把email作为唯一标识，

3）紧接着调用userService的selectByEmail()方法，通过email查找用户

4）若查询不到，则说明没有这个用户，抛出UnknownAccountException异常。若用户状态无效，则账号被禁止使用，抛出LockedAccountException异常。然后返回`new SimpleAuthenticationInfo(user, user.getPswd(), getName())`

5）现在shiro就开始密码匹配了，user.getPswd()传递的是数据库中正确的密码，而前端传递的密码shiro则会通过继承了UsernamePasswordToken的类的getPassword()获取，这些过程不需要我们自己实现，shiro会帮助我们进行

5.1）需要注意的是，如果在spring-shiro.xml配置了credentialsMatcher

```xml
<!-- 授权 认证 -->
<bean id="simpleRealm" class="com.hemou.core.shiro.token.SimpleRealm">
    <property name="credentialsMatcher">
        <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
            <!--加密算法-->
            <property name="hashAlgorithmName" value="MD5"/>
            <!--加密次数-->
            <property name="hashIterations" value="1024"/>
        </bean>
    </property>
</bean>
```

在这里我是用的MD5加密算法，并迭代了1024次，==所以当我们注册用户时我们也要用同样的算法并迭代1024次，再存入数据库中==

5.2）如果匹配错误，则会抛出IncorrectCredentialsException异常，即密码错误

4.2、如果前面抛出了异常，则通过try catch就可以捕获这些异常，并返回错误信息

AccountController.java

```java
@RequestMapping("u")
@Controller
public class AccountController extends BaseController {
          
    @ResponseBody
    @PostMapping("submitLogin")
    public Object submitLogin(boolean rememberMe, UUser entity, HttpServletRequest request){
        try {
            UUser user = TokenManager.login(entity, rememberMe);
			...
        } catch (UnknownAccountException e){
            return Result.error(e.getMessage());
        } catch (IncorrectCredentialsException e) {
            return Result.error("密码错误！");
        } catch (LockedAccountException e){
            return Result.error(e.getMessage());
        }
    }
```

4.3、如果没有发生错误，则不会抛出异常

```java
@RequestMapping("u")
@Controller
public class AccountController extends BaseController {
  
    @ResponseBody
    @PostMapping("submitLogin")
    public Object submitLogin(boolean rememberMe, UUser entity, HttpServletRequest request){
        try {
            UUser user = TokenManager.login(entity, rememberMe);

            // 更新最后登录时间
            user.setLastLoginTime(new Date());
            userService.updateByPrimaryKeySelective(user);

            SavedRequest savedRequest = WebUtils.getSavedRequest(request);
            String url = null;
            if(null != savedRequest){
                url = savedRequest.getRequestUrl();
            }
            LoggerUtils.info(getClass(), "submitLogin：之前的访问路径 [%s]", url);
            if(StringUtils.isEmpty(url)) url = "user/index.shtml";
            return Result.success("登录成功！", url);
        }catch...
    }
```

我们这一在这里修改最后登录时间，然后通过`WebUtils.getSavedRequest(request);`获取登录之前的路径，以便成功登录后回到以前

### 注册

1、访问登录页面 localhost:8080/shiro/u/register.shtml

2、spring-mvc拦截以shtml结尾的请求，转到register.ftl

AccountController.java

```java
@RequestMapping("u")
@Controller
public class AccountController extends BaseController {
    @GetMapping("register")
    public String toRegister(){
        return "common/register";
    }
```

3、在register.ftl填写完表单后，发送请求[post] localhost:8080/shiro/u/submitRegister.shtml

4、springmvc拦截后来到submitRegister方法

AccountController.java

```java
@ResponseBody
@PostMapping("submitRegister")
public Object submitRegister(String verifyCode, UUser entity){
    if(!TokenManager.isVCodeValid(verifyCode)) return Result.warning("验证码无效或有误！");

    // 清除验证码
    TokenManager.clearVerifyCode();

    // 验证email是否存在
    String email = entity.getEmail();
    UUser user = userService.selectByEmail(email);
    if(user != null) return Result.warning("邮箱已存在请重新注册！");

    // 插入数据
    Date date = new Date();
    String pswd = entity.getPswd();
    String password = EncryptionUtil.encryptionPassword(pswd);
    entity.setPswd(password);
    entity.setStatus(UUser.VALID);
    entity.setCreateTime(date);
    entity.setLastLoginTime(date);
    userService.insertSelective(entity);
    LoggerUtils.info(getClass(), "注册：成功插入数据 [%s]", entity);

    // 登录
    entity.setPswd(pswd);
    entity = TokenManager.login(entity, true);
    LoggerUtils.info(getClass(), "[%s]注册成功！", entity);
    return Result.success("注册成功", "user/index.shtml");
}
```

1）先验证验证码是否有效，以及清除

2）然后再通过查询判断邮箱是否存在

3）然后插入数据，需要注意的是插入到数据库的中的数据是加密过的，所以我们要用EncryptionUtil.encryptionPassword进行加密

EncryptionUtil.java

```java
public class EncryptionUtil {
    public static String encryptionPassword(String credentials){
        String hashAlgorithmName = "MD5";
        int hashIterations = 1024;
        return String.valueOf(new SimpleHash(hashAlgorithmName, credentials, null, hashIterations));
    }
```

因为之前在spring-shiro.xml中配置的是MD5算法并迭代1024次，所以这里任然用同样的方法加密

spring-shiro.xml

```xml
<!-- 授权 认证 -->
<bean id="simpleRealm" class="com.hemou.core.shiro.token.SimpleRealm">
  <property name="credentialsMatcher">
    <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
      <!--加密算法-->
      <property name="hashAlgorithmName" value="MD5"/>
      <!--加密次数-->
      <property name="hashIterations" value="1024"/>
    </bean>
  </property>
</bean>
```

4）使用shiro登录，并设置rememberMe为true，这里需要注意的是，因为使用shiro的login方法时，他会自动将前端的密码加密后再与数据中的密码进行校验，所以这里我们要把entity中的密码恢复成前端明文密码

5）注册成功，返回Result.success("注册成功", "user/index.shtml");

### 个人中心

#### 信息展示

1、登录成功，通过js代码 window.location.href= result.obj 来到用户信息展示页

login.ftl  部分js

```js
$.operate.post('${basePath}/u/submitLogin.shtml', {email: username, pswd: password, rememberMe: check}, function (result) {
    if(result && result.status !== resp_status.SUCCESS){
        $('#password').val('');
    }else{
        setTimeout(function(){
            window.location.href= result.obj || "${basePath}/";
        },1500)
    }
})
```

而 result.obj 则是由后端传递的用户信息展示页地址的信息

AccountController.java 部分

```java
@ResponseBody
@PostMapping("submitLogin")
public Object submitLogin(boolean rememberMe, UUser entity, HttpServletRequest request){
    try {
        ...
        String url = null;
        if(null != savedRequest){
            url = savedRequest.getRequestUrl();
        }
        LoggerUtils.info(getClass(), "submitLogin：之前的访问路径 [%s]", url);
        if(StringUtils.isEmpty(url)) url = "user/index.shtml";
        return Result.success("登录成功！", url);
    } catch ....
}
```

Result.java 部分

```java
public static Result success(String message, Object obj){
    return new Result(HTTPConstant.REQUEST_SUCCESS, message, obj);
}
```

2、来看 user/index.shtml 部分内容

```html
<h2>个人资料</h2><hr>
<table class="table table-bordered">
    <tr>
        <th>昵称</th>
        <td>${token.nickname?default('未设置')}</td>
    </tr>
    <tr>
        <th>Email/帐号</th>
        <td>${token.email?default('未设置')}</td>
    </tr>
    <tr>
        <th>创建时间</th>
        <td>${token.createTime?string('yyyy-MM-dd HH:mm')}</td>
    </tr>
    <tr>
        <th>最后登录时间</th>
        <td>${token.lastLoginTime?string('yyyy-MM-dd HH:mm')}</td>
    </tr>
</table>
```

这里token我们会在许多地方用到，所以我们要通过配置FreeMarkerView，将token的值纳入模型数据中，前面关于freemarker的环境配置也有讲到

FreemarkerViewExtend.java

```java
public class FreemarkerViewExtend extends FreeMarkerView {
    @Override
    protected void exposeHelpers(Map<String, Object> model, HttpServletRequest request) throws Exception {
        super.exposeHelpers(model, request);
        if(TokenManager.isLogin())model.put("token", TokenManager.getToken());
        model.put("basePath", request.getContextPath());
    }    
}
```

TokenManager.java

```java
public class TokenManager {
    public static UUser getToken(){
        return (UUser) getSubject().getPrincipal();
    }
    public static boolean isLogin(){
        return null != getSubject().getPrincipal();
    }
    ...
```

这样我们就可以通过 ${token.xxx}的方式渲染个人信息了

#### 修改信息

1、前端填写完数据发送请求 [post] localhost:8080/shiro/user/update.shtml

2、springmvc拦截，来到update方法

```java
@RestController
@RequestMapping("user")
public class UserController extends BaseController {
    @Resource
    private UUserService userService;
    @PostMapping("update")
    public Object update(UUser entity){
        String nickname = entity.getNickname();
        if(StringUtils.isEmpty(nickname)) return Result.warning("昵称不可为空！");

        int update = userService.updateByPrimaryKeySelective(entity);
        LoggerUtils.info(getClass(), "修改用户 [%s] %s", entity, update==1 ? "成功！" : "失败");
        if(update == 1){
            UUser user = userService.selectByPrimaryKey(entity.getId());
            TokenManager.setUser(user);
            return Result.success("修改成功！");
        }else{
            return Result.error("修改失败！");
        }
    }
```

唯一要注意的地方就是当我们修改完昵称后，而shiro的subject还是原来的信息，这时有两种方法

第一种就是通过shiro的login方法重新登录一次，显然这个不可取（因为shiro的登录需要明文密码，而由shiro的方法SecurityUtils.getSubject().getPrincipal()已经是从数据库中获取的加密过的密码了）

SimpleRealm.java

```java
public class SimpleRealm extends AuthorizingRealm {
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        ShiroToken token = (ShiroToken)authenticationToken;
        UUser user = userService.selectByEmail(token.getUsername());
        ...
        return new SimpleAuthenticationInfo(user, user.getPswd(), getName());
    }
}
```

第二种重新装载suject信息

TokenManager.java

```java
public static void setUser(UUser user){
    Subject subject = getSubject();
    PrincipalCollection principalCollection = getSubject().getPrincipals();
    String realmName = principalCollection.getRealmNames().iterator().next();
    PrincipalCollection newPrincipalCollection =
        new SimplePrincipalCollection(user, realmName);
    subject.runAs(newPrincipalCollection);
}
```

通过调用TokenManager.setUser(user); 即使刷新前端页面也可以发现信息已近修改过来了



## 错误记录

**修改个人信息**

[前端错误代码]

```js
$(function () {
    if($.common.isEmpty($('#nickname').val())){
        $.modal.msgError('昵称不能为空！')
        $("#nickname").parent().removeClass('has-success').addClass('has-error')
    }else{
        $.operate.post('${basePath}/user/update.shtml', {nickname}, function (result) {
            setTimeout(function () {
                $.modal.reload()
            }, 1000)
        })
    }
});
```

错误1：$(function () {}) 中的代码会立即执行，外边应该包一层事件监听，不知道当时咋想的:cold_sweat:

错误2：nickname没有申明，也没有赋值，而且nickname在这个项目貌似是个特殊的变量名，浏览器居然没检测出他未申明，还有这里用了es6新语法，而nickname未赋值，所以控制台一直报Uncaught RangeError: Maximum call stack size exceeded异常（猜测）

错误3：修改信息需要带上一个id信息，不过这个项目中可以在后端通过token获取用户信息，也算个小错误吧

注意：form里面的button标签点击一次，不管form的action是否有值，都会发送一次请求，为避免应使用input设置type为button这样可以避免发送请求，或者通过js阻止默认事件

[前端纠正代码]

```js
$(function () {
    $('input[type=button]').click(function () {
        let id = $('#id').val()
        let nickname = $('#nickname').val()
        if($.common.isEmpty(nickname)){
            $.modal.msgError('昵称不能为空！')
            $("#nickname").parent().removeClass('has-success').addClass('has-error')
        }else{
            $.operate.post('${basePath}/user/update.shtml', {id, nickname}, function (result) {
                setTimeout(function () {
                    $.modal.reload()
                }, 1000)
            })
        }
    })
});
```

**com.github.pagehelper.PageInterceptor插件空指针异常**

刚开始是这么配置的

```xml
<property name="plugins">
    <array>
        <bean class="com.github.pagehelper.PageInterceptor"/>
    </array>
</property>
```

改成这样就没事了

```xml
<property name="plugins">
    <array>
        <bean class="com.github.pagehelper.PageInterceptor">
            <property name="properties">
                <value/>
            </property>
        </bean>
    </array>
</property>
```

**mybatis基本类型非空判断**

mybatis 出现异常：There is no getter for property...

见参考



## 参考

[项目原始来源](https://www.sojson.com/shiro)

[No converter found for return value of type 异常问题](https://blog.csdn.net/qq_35917800/article/details/78452938)

[shiro前端密码加密问题](https://www.bilibili.com/video/BV1YW411M7S3?p=11)

[ehcache配置文件说明](https://blog.csdn.net/u011955525/article/details/106970207/)

[freemarker相关](https://www.cnblogs.com/peijie-tech/articles/11831262.html)

[shiro-freemarker标签](https://www.sojson.com/blog/143.html)

[shiro过滤器工作原理](https://blog.csdn.net/achenyuan/article/details/78541529)

[shiro更改subject](https://blog.csdn.net/cpohui/article/details/106853932)

[Mybatis基本类型参数非空判断](https://www.cnblogs.com/zxxfz/p/11987980.html)



