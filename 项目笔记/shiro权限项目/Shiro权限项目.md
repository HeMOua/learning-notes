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

  <!-- 引入属性文件 -->
  <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="ignoreResourceNotFound" value="true"/>
    <property name="locations">
      <list>
        <value>classpath:jdbc.properties</value>
        <value>classpath:shiro-config.properties</value>
      </list>
    </property>
  </bean>
    
  <import resource="spring-mybatis.xml"/>
  <import resource="spring-shiro.xml"/>

</beans>
```

==注意==：

1、这里引入properties文件不能使用\<context:property-placeholder/>，Spring容器仅允许最多定义一个property-placeholder，其余的会被Spring忽略掉。

2、locations的value值要加上`classpath:`，否则可能会出现FileNotFoundException: Could not open ServletContext resource [/jdbc.properties]异常

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

### 授权

配置：在spring-shiro.xml配置中

```xml
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    ...
    <property name="filterChainDefinitions" >
      <value>
        /u/** = anon
        /js/** = anon
        /css/** = anon
        /img/** = anon
        /user/** = user
        /member/** = roles[admin]
        /** = user
      </value>
    </property>
  </bean>
```

这表示当访问 /member/** 时必须有 admin 这个角色

过程分析

1、当访问 localhost:8080/shiro/member/index.shtml时，会被roles这个拦截器拦截，其实现类为RolesAuthorizationFilter.java，观其源码

RolesAuthorizationFilter.java

```java
public class RolesAuthorizationFilter extends AuthorizationFilter {
    public boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws IOException {
        Subject subject = this.getSubject(request, response);
        String[] rolesArray = (String[])((String[])mappedValue);
        if (rolesArray != null && rolesArray.length != 0) {
            Set<String> roles = CollectionUtils.asSet(rolesArray);
            return subject.hasAllRoles(roles);
        } else {
            return true;
        }
    }
}
```

它会调用`subject.hasAllRoles(roles);`，这个方法呢会接着调用 `getAuthorizationInfo() `方法

```java
public boolean hasAllRoles(PrincipalCollection principal, Collection<String> roleIdentifiers) {
    AuthorizationInfo info = this.getAuthorizationInfo(principal);
    return info != null && this.hasAllRoles(roleIdentifiers, info);
}
```

而 `getAuthorizationInfo() `方法会调用 doGetAuthorizationInfo() 方法，也就是我们自己写的授权方法

AuthorizingRealm.java  省略了部分打印日志方法

```java
protected AuthorizationInfo getAuthorizationInfo(PrincipalCollection principals) {
    if (principals == null) {
        return null;
    } else {
        AuthorizationInfo info = null;
        ...
        Cache<Object, AuthorizationInfo> cache = this.getAvailableAuthorizationCache();
        Object key;
        if (cache != null) {
            ...
            key = this.getAuthorizationCacheKey(principals);
            info = (AuthorizationInfo)cache.get(key);
            ...
        }
        if (info == null) {
            info = this.doGetAuthorizationInfo(principals);
            if (info != null && cache != null) {
                ...
                key = this.getAuthorizationCacheKey(principals);
                cache.put(key, info);
            }
        }
        return info;
    }
}
```

2、在doGetAuthorizationInfo() 方法中，我们通过查询数据库赋予角色相应的角色和权限，根据上面的源码可知，此方法只会调用一次，之后都会从缓存中获取授权信息

```java
public class SimpleRealm extends AuthorizingRealm {
	...
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        UUser user = TokenManager.getUser();
        // 拥有的角色
        Set<String> roleSet = new HashSet<>();
        List<URole> roles = roleService.selectByUserId(user.getId());
        for(URole r : roles) roleSet.add(r.getType());
        info.setRoles(roleSet);
        // 拥有的权限
        Set<String> permsSet = permissionService.selectByUserId(user.getId());
        info.setStringPermissions(permsSet);
        return info;
    }
```

**拓展roles过滤器**

当配置 `/member/** = roles["admin,user"]`时(注：当有两个角色时，需要加双引号)，shiro自带的roles过滤器会判断是否拥有admin和user这个两个角色，而往往我们能只要拥有之中的一个就行，所以我们要自定义一个过滤器满足我们需要的要求

RoleOrFilter.java

```java
public class RoleOrFilter extends AuthorizationFilter {
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        Subject subject = this.getSubject(request, response);
        String[] rolesArray = (String[])mappedValue;
        if (rolesArray != null && rolesArray.length != 0) {
            for(String role : rolesArray){
                if(subject.hasRole(role)) return true;
            }
            return false;
        } else {
            return true;
        }
    }
}
```

spring-shiro.xml

```xml
<bean id="roleOrFilter" class="com.hemou.core.shiro.filter.RoleOrFilter"/>
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
  ...
  <property name="filters">
    <util:map>
      <entry key="roleOr" value-ref="roleOrFilter"/>
    </util:map>
  </property>
  <!-- 初始配置，现采用自定义  -->
  <property name="filterChainDefinitions" >
    <value>
      ...
      /member/** = roleOr["admin,user"]
      /** = user
    </value>
  </property>
</bean>
```

这样当我们拥有 admin或user 其中之一的角色就可以正常访问了

### 查看在线成员并踢出

#### 配置

1、这一块相当复杂，首先看配置文件

spring-shiro.xml

```xml
<!-- 安全管理器 -->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="realm" ref="simpleRealm"/>
    <property name="sessionManager" ref="sessionManager"/>
    <property name="rememberMeManager" ref="rememberMeManager"/>
    <property name="cacheManager" ref="cacheManager"/>
</bean>
<!-- Session Manager -->
<bean id="sessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
    <!-- 相隔多久检查一次session的有效性   -->
    <property name="sessionValidationInterval" value="${session.validate.timespan}"/>
    <!-- session 有效时间为半小时 （毫秒单位）-->
    <property name="globalSessionTimeout" value="${session.timeout}"/>
    <property name="sessionDAO" ref="customShiroSessionDAO"/>
    <!-- 间隔多少时间检查，不配置是60分钟 -->
    <property name="sessionValidationScheduler" ref="sessionValidationScheduler"/>
    <!-- 是否开启 检测，默认开启 -->
    <property name="sessionValidationSchedulerEnabled" value="true"/>
    <!-- 是否删除无效的，默认也是开启 -->
    <property name="deleteInvalidSessions" value="true"/>
    <!-- 会话Cookie模板 -->
    <property name="sessionIdCookie" ref="sessionIdCookie"/>
</bean>

<!-- 会话Session ID生成器 -->
<bean id="sessionIdGenerator" class="org.apache.shiro.session.mgt.eis.JavaUuidSessionIdGenerator"/>

<!-- 会话验证调度器 -->
<bean id="sessionValidationScheduler" class="org.apache.shiro.session.mgt.ExecutorServiceSessionValidationScheduler">
    <!-- 间隔多少时间检查，不配置是60分钟 -->
    <property name="interval" value="${session.validate.timespan}"/>
    <property name="sessionManager" ref="sessionManager"/>
</bean>

<!-- 会话Cookie模板 -->
<bean id="sessionIdCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
    <constructor-arg value="shiro-sessionCookie"/>
    <property name="httpOnly" value="true"/>
    <!--cookie的有效时间 -->
    <property name="maxAge" value="-1"/>
</bean>
```

这里我们把前面注解掉的sessionManager放开，下面就是要配置sessionManager了

在sessionManager配置中最重要的是 sessionDAO，这就要我们自定义的一个Dao来查询session

2、自定义sessionDao

自定义sessionDao需要继承 AbstractSessionDAO 类，并完成他的抽象方法

CustomShiroSessionDao.java

```java
public class CustomShiroSessionDAO extends AbstractSessionDAO {
    
    private ShiroSessionRepository shiroSessionRepository;
    
    @Override
    protected Serializable doCreate(Session session) {
        return null;
    }
    
    @Override
    protected Session doReadSession(Serializable serializable) {
        return null;
    }
    
    @Override
    public void update(Session session) throws UnknownSessionException {    }
    
    @Override
    public void delete(Session session) {    }
    
    @Override
    public Collection<Session> getActiveSessions() {
        return null;
    }
    。。。省略get、set
}
```

3、实现上面的空方法，也就是操作session不是一句两句就能实现的，所以我们先新建一个ShiroSessionRepository接口，抽象操作session的相关方法

```java
public interface ShiroSessionRepository {
    /**
     * 存储Session
     * @param session
     */
    void saveSession(Session session);
    /**
     * 删除session
     * @param sessionId
     */
    void deleteSession(Serializable sessionId);
    /**
     * 获取session
     * @param sessionId
     * @return
     */
    Session getSession(Serializable sessionId);
    /**
     * 获取所有sessoin
     * @return
     */
    Collection<Session> getAllSessions();
}
```

4、在这个项目中我们使用redis进行缓存，所以我们新建一个redis操作session的实现类JedisShiroSessionRepository

JedisShiroSessionRepository.java

```java
public class JedisShiroSessionRepository implements ShiroSessionRepository {

    private JedisManager jedisManager;

    @Override
    public void saveSession(Session session) {        
    }

    @Override
    public void deleteSession(Serializable id) {
    }

    @Override
    public Session getSession(Serializable id) {
        ...
    }

    @Override
    public Collection<Session> getAllSessions() {
        ...
    }

    private String buildRedisSessionKey(Serializable sessionId) {
        return ShiroConstant.SHIRO_SESSION_TAG + sessionId;
    }

    public JedisManager getJedisManager() {
        return jedisManager;
    }

    public void setJedisManager(JedisManager jedisManager) {
        this.jedisManager = jedisManager;
    }
}
```

5、在上面这个类中，我们就要实打实的用redis来操作并存储数据了，所以我们还要新建一个JedisManager来操作redis，也就是操作redis的dao。

在新建JedisManger之前，先想想需要哪些操作

1. 首先获取redis连接是必须要有的，当然还有释放redis连接
2. 其次思考一下使用redis中的什么数据类型。因为我们可以将对象序列化成字符，所以我们可以选用 string 类型操作数据
3. 接下来就添加、删除和获取这三个方法了

现在来实现JedisManger

JedisManger.java

```java
public class JedisManager {

    private JedisPool jedisPool;

    public Jedis getJedis() {
        Jedis jedis = null;
        try {
            jedis = getJedisPool().getResource();
            return jedis;
        } catch (JedisConnectionException e) {
            LoggerUtils.error(getClass(), "尚未启动redis，系统自动退出");
            System.exit(0);
            throw new JedisConnectionException(e);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 获取键值
     * @param dbIndex
     * @param key
     * @return
     * @throws Exception
     */
    public byte[] get(int dbIndex, byte[] key) throws Exception {
        Jedis jedis = null;
        byte[] result = null;
        try {
            jedis = getJedis();
            jedis.select(dbIndex);
            result = jedis.get(key);
            return result;
        } finally {
            returnResource(jedis);
        }
    }

    /**
     * 删除键
     * @param dbIndex
     * @param key
     * @throws Exception
     */
    public void del(int dbIndex, byte[] key) throws Exception {
        Jedis jedis = null;
        boolean isBroken = false;
        try {
            jedis = getJedis();
            jedis.select(dbIndex);
            jedis.del(key);
        } finally {
            returnResource(jedis);
        }
    }

    /**
     * 设置值和到期时间
     * @param dbIndex
     * @param key
     * @param value
     * @param expireTime 如果 expireTime > 0 才设置
     * @throws Exception
     */
    public void setex(int dbIndex, byte[] key, byte[] value, int expireTime) throws Exception {
        Jedis jedis = null;
        try {
            jedis = getJedis();
            jedis.select(dbIndex);
            jedis.set(key, value);
            if (expireTime > 0) jedis.expire(key, expireTime);
        } finally {
            returnResource(jedis);
        }
    }

    public Collection<Session> getAllSession(int dbIndex, String redisShiroSession) throws Exception {
        Jedis jedis = null;
        Set<Session> sessions = new HashSet<Session>();
        try {
            jedis = getJedis();
            jedis.select(dbIndex);
            Set<byte[]> byteKeys = jedis.keys((ShiroConstant.SHIRO_SESSION_ALL).getBytes());
            if (byteKeys != null && byteKeys.size() > 0) {
                for (byte[] bs : byteKeys) {
                    Session obj = ObjectUtil.deserialize(jedis.get(bs));
                    if(obj != null) sessions.add(obj);
                }
            }
            return sessions;
        } finally {
            returnResource(jedis);
        }
    }

    /**
     * 归还资源
     * @param jedis
     */
    public void returnResource(Jedis jedis) {
        if (jedis == null) return;
        jedis.close();
    }

    public JedisPool getJedisPool() {
        return jedisPool;
    }

    public void setJedisPool(JedisPool jedisPool) {
        this.jedisPool = jedisPool;
    }
}
```

6、有了JedisManager，接下来我们就可以继续实现 JedisShiroSessionRepository 类了

不过在此之前先介绍一下所有的常量

ShiroConstant.java

```java
public class ShiroConstant {

    /**
     * 数据库
     */
    public static final int DB_INDEX = 0;
    public static final int SESSION_EXPIRE_TIME = 1800;

    /**
     * 项目 session 标记
     */
    public static final String SHIRO_SESSION_TAG = "shiro-session:";

    /**
     * 查询 session 表达式
     */
    public static final String SHIRO_SESSION_ALL = "*shiro-session:*";

    /**
     * 在线状态属性名
     */
    public static final String ONLINE_STATUS_KEY = "shiro-online-status";

    /**
     * 踢出后到达页面
     */
    public static final String KICK_OUT_URL = "/u/kickout.shtml";

    /**
     * 踢出状态属性名
     */
    public static final String KICK_OUT_STATUS_KEY = "shiro-kick-out";

    /**
     * 在线状态标记
     */
    public static final String ONLINE_STATUS_TAG = "shiro-online:";
}
```

然后我们接着实现JedisShiroSessionRepository 这个类

首先解释在这个类中用到的三个常量的的作用

+ SHIRO_SESSION_TAG，他就是我们存入redis数据的一个标签，以他开头的键名，表示其键值是与session有关的数据
+ SHIRO_SESSION_ALL，这个是用来查询所有带SHIRO_SESSION_TAG这个标签键名的数据
+ DB_INDEX，表示操作的是redis的那个数据库

下面是JedisShiroSessionRepository操作的实现

JedisShiroSessionRepository.java

```java
public class JedisShiroSessionRepository implements ShiroSessionRepository {

    private JedisManager jedisManager;

    @Override
    public void saveSession(Session session) {
        if (session == null || session.getId() == null) 
            throw new NullPointerException("session is empty");
        try {
            // 序列化session key
            byte[] key = ObjectUtil.serialize(buildRedisSessionKey(session.getId()));
            // 如果不存在状态就设置
            if(null == session.getAttribute(ShiroConstant.ONLINE_STATUS_KEY)){
                session.setAttribute(ShiroConstant.ONLINE_STATUS_KEY, true);
            }
            // 序列化session value
            byte[] value = ObjectUtil.serialize(session);
            getJedisManager().setex(ShiroConstant.DB_INDEX, key, value, 
                                    (int) (session.getTimeout() / 1000));
        } catch (Exception e) {
            e.printStackTrace();
            LoggerUtils.error(getClass(), "save session error，id:[%s]",session.getId());
        }
    }

    @Override
    public void deleteSession(Serializable id) {
        if (id == null) throw new NullPointerException("session id is empty");
        try {
            getJedisManager().del(ShiroConstant.DB_INDEX,
                                  ObjectUtil.serialize(buildRedisSessionKey(id)));
        } catch (Exception e) {
            e.printStackTrace();
            LoggerUtils.error(getClass(), "删除session出现异常，id:[%s]",id);
        }
    }

    @Override
    public Session getSession(Serializable id) {
        if (id == null) throw new NullPointerException("session id is empty");
        Session session = null;
        try {
            byte[] value = getJedisManager().get(ShiroConstant.DB_INDEX,
                                                 ObjectUtil.serialize(buildRedisSessionKey(id)));
            session = ObjectUtil.deserialize(value);
        } catch (Exception e) {
            e.printStackTrace();
            LoggerUtils.error(getClass(), "获取session异常，id:[%s]",id);
        }
        return session;
    }

    @Override
    public Collection<Session> getAllSessions() {
        Collection<Session> sessions = null;
        try {
            sessions = getJedisManager().getAllSession(ShiroConstant.DB_INDEX,
                                                       ShiroConstant.SHIRO_SESSION_TAG);
        } catch (Exception e) {
            e.printStackTrace();
            LoggerUtils.error(getClass(), "获取全部session异常");
        }
        return sessions;
    }

    private String buildRedisSessionKey(Serializable sessionId) {
        return ShiroConstant.SHIRO_SESSION_TAG + sessionId;
    }

    public JedisManager getJedisManager() {
        return jedisManager;
    }

    public void setJedisManager(JedisManager jedisManager) {
        this.jedisManager = jedisManager;
    }
}
```

7、到了这步我们就可以完成第2步所说的sessionDao了，下面就是CustomShiroSessionDAO的完整代码

CustomShiroSessionDAO.java

```java
public class CustomShiroSessionDAO extends AbstractSessionDAO {

    private ShiroSessionRepository shiroSessionRepository;

    @Override
    protected Serializable doCreate(Session session) {
        Serializable sessionId = this.generateSessionId(session);
        this.assignSessionId(session, sessionId);
        getShiroSessionRepository().saveSession(session);
        return sessionId;
    }

    @Override
    protected Session doReadSession(Serializable sessionId) {
        return getShiroSessionRepository().getSession(sessionId);
    }

    @Override
    public void update(Session session) throws UnknownSessionException {
        getShiroSessionRepository().saveSession(session);
    }

    @Override
    public void delete(Session session) {
        if (session == null) {
            LoggerUtils.error(getClass(), "Session 不能为null");
            return;
        }
        Serializable id = session.getId();
        if (id != null) getShiroSessionRepository().deleteSession(id);
    }

    @Override
    public Collection<Session> getActiveSessions() {
        return getShiroSessionRepository().getAllSessions();
    }

    public ShiroSessionRepository getShiroSessionRepository() {
        return shiroSessionRepository;
    }

    public void setShiroSessionRepository(ShiroSessionRepository shiroSessionRepository) {
        this.shiroSessionRepository = shiroSessionRepository;
    }
}
```

8、最后在回到spring-shiro.xml，这样我们的sessionDAO就配置好了

spring-shiro.xml

```xml
<bean id="sessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
	...
    <property name="sessionDAO" ref="customShiroSessionDAO"/>
    ...
</bean>
<!-- custom shiro session dao -->
<bean id="customShiroSessionDAO" class="com.hemou.core.shiro.session.CustomShiroSessionDAO">
  <property name="shiroSessionRepository" ref="jedisShiroSessionRepository"/>
  <property name="sessionIdGenerator" ref="sessionIdGenerator"/>
</bean>
<!-- 会话Session ID生成器 -->
<bean id="sessionIdGenerator" class="org.apache.shiro.session.mgt.eis.JavaUuidSessionIdGenerator"/>
```

这样由redis存储并操作session所必须的配置都完成了

#### 查看在线成员

1、查看在线成员，我们需要知道用户本身的信息及其如下的，我们将这些信息封装起来

UserOnlineBo.java

```java
public class UserOnlineBo extends UUser implements Serializable {

    //Session Id
    private String sessionId;
    //Session Host
    private String host;
    //Session创建时间
    private Date startTime;
    //Session最后交互时间
    private Date lastAccess;
    //Session timeout
    private long timeout;
    //session 状态
    private boolean sessionStatus = true;

    public UserOnlineBo() {
    }

    public UserOnlineBo(UUser user) {
        super(user);
    }
```

2、之前编写的类都是配置所必须的，但是操作session仅用上面所编写的类完全不够，因此我们还要在写一个CustomSessionManager，编写操作session的其他方法

CustomSessionManager.java

```java
public class CustomSessionManager {

    private ShiroSessionRepository shiroSessionRepository;
    private CustomShiroSessionDAO customShiroSessionDAO;

    /**
     * 获取所有的有效Session用户
     *
     * @return
     */
    public List<UserOnlineBo> getAllUserOnline() {
        //获取所有session
        Collection<Session> sessions = customShiroSessionDAO.getActiveSessions();
        List<UserOnlineBo> list = new ArrayList<>();
        for (Session session : sessions) {
            UserOnlineBo bo = getSessionBo(session);
            if (null != bo) {
                list.add(bo);
            }
        }
        return list;
    }
    
    private UserOnlineBo getSessionBo(Session session) {
        //获取session登录信息。
        Object obj = session.getAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY);
        if (null == obj) {
            return null;
        }
        //确保是 SimplePrincipalCollection对象。
        if (obj instanceof SimplePrincipalCollection) {
            SimplePrincipalCollection spc = (SimplePrincipalCollection) obj;
            obj = spc.getPrimaryPrincipal();
            if (null != obj && obj instanceof UUser) {
                //存储session + user 综合信息
                UserOnlineBo userBo = new UserOnlineBo((UUser) obj);
                //最后一次和系统交互的时间
                userBo.setLastAccess(session.getLastAccessTime());
                //主机的ip地址
                userBo.setHost(session.getHost());
                //session ID
                userBo.setSessionId(session.getId().toString());
                //session最后一次与系统交互的时间
                userBo.setLastLoginTime(session.getLastAccessTime());
                //回话到期 ttl(ms)
                userBo.setTimeout(session.getTimeout());
                //session创建时间
                userBo.setStartTime(session.getStartTimestamp());
                //是否踢出
                Boolean onlineStatus = (Boolean) session.getAttribute(ShiroConstant.ONLINE_STATUS_KEY);
                userBo.setSessionStatus(null == onlineStatus ? true : onlineStatus);
                return userBo;
            }
        }
        return null;
    }
```

先将之前编写的操作session的类注入进来，然后调用获取所有session的方法[customShiroSessionDAO.getActiveSessions()]，这样就可以查询在线用户了

3、编写controller

```java
@Controller
@RequestMapping("member")
public class MemberController extends BaseController {

	@Autowired
    private CustomSessionManager sessionManager;    
    
    @GetMapping("online")
    public String online(Model model){
        List<UserOnlineBo> users = sessionManager.getAllUserOnline();
        model.addAttribute("users", users);
        return "member/online";
    }
```

然后编写个页面前端展示一下就ok了

#### 踢出用户

0、先看一下之前创建session的代码

CustomShiroSessionDAO.java

```java
public class CustomShiroSessionDAO extends AbstractSessionDAO {
    @Override
    protected Serializable doCreate(Session session) {
        Serializable sessionId = this.generateSessionId(session);
        this.assignSessionId(session, sessionId);
        getShiroSessionRepository().saveSession(session);
        return sessionId;
    }
```

创建session时会调用saveSession方法。那么接着看一下saveSession方法

JedisShiroSessionRepository.java

```java
public class JedisShiroSessionRepository implements ShiroSessionRepository {
    @Override
    public void saveSession(Session session) {
        ...
        try {
            // 序列化session key
            byte[] key = ...
            // 如果不存在状态就设置
            if(null == session.getAttribute(ShiroConstant.ONLINE_STATUS_KEY)){
                session.setAttribute(ShiroConstant.ONLINE_STATUS_KEY, true);
            }
            // 序列化session value
            byte[] value = ObjectUtil.serialize(session);
            getJedisManager().setex( ...)
        } catch (Exception e) {....
    }
```

由此可以看出创建session时我们添加了一个标记 ShiroConstant.ONLINE_STATUS_KEY，来表示用户是否在线

1、自定义一个shiro的filter

```java
public class OnlineFilter extends AccessControlFilter {
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object o) 
        throws Exception {
        Subject subject = getSubject(request, response);
        Session session = subject.getSession();

        Boolean isOnline = (Boolean) session.getAttribute(ShiroConstant.ONLINE_STATUS_KEY);
        if(null != isOnline && !isOnline){ 
            return false; // 如果 isOnline = false，则拦截，前往onAccessDenied方法
        }
        return true;
    }

    @Override
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response) 
        throws Exception {
        Subject subject = getSubject(request, response);
        subject.logout();
        saveRequest(request);
        if(HTTPUtils.isAjax(request)){
            response.setCharacterEncoding("UTF-8");
            Result error = Result.error("您已经被踢出，请重新登录或联系管理员！");
            error.setObj("reload");
            HTTPUtils.out(response, error);
        }else{
            WebUtils.issueRedirect(request, response, ShiroConstant.KICK_OUT_URL);
        }
        return false;
    }
}
```

在onAccessDenied方法中，先退出登录，然后判断请求是否是ajax

如果是，则返回一个json数据，我在这里设置了一个标记 reload，这样前端接收到数据后会自动刷新当前页面，而之前调用了subject.logout()方法，没有了登录凭证，就会自动来到登录页面（由之前的shiro配置）

 如果不是，则重定向到 踢出页面（此页面无需登录凭证）

2、配置加载OnlineFilter

```xml
<bean id="onlineFilter" class="com.hemou.core.shiro.filter.OnlineFilter"/>

<!--shiro 过滤器配置-->
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
	...
    <property name="filters">
        <util:map>
            ...
            <entry key="online" value-ref="onlineFilter"/>
        </util:map>
    </property>
    <!--	初始配置，现采用自定义	-->
    <property name="filterChainDefinitions" >
        <value>
            ...
            /user/** = online
            /member/** = roleOr["admin,user"],online
            /permission/** = roleOr["admin,perms"],online
            /role/** = roleOr["admin,role"],online
            /** = user
        </value>
    </property>
</bean>
```

3、修改用户状态。在CustomSessionManager新增一个方法

CustomSessionManager.java

```java
/**
 * 改变Session状态
 *
 * @param status     {true:踢出,false:激活}
 * @param sessionIds
 * @return
 */
public Result changeSessionStatus(boolean status, String sessionIds) {
    try {
        Session session = shiroSessionRepository.getSession(sessionIds);
        session.setAttribute(ShiroConstant.ONLINE_STATUS_KEY, status);
        customShiroSessionDAO.update(session);
        String msg = "成功" + (status ? "激活" : "踢掉") + "用户！";
        return Result.success(msg);
    } catch (Exception e) {
        e.printStackTrace();
        LoggerUtils.error(getClass(), "改变Session状态错误，sessionId[%s]", sessionIds);
        return Result.error("改变失败，有可能Session不存在，请刷新再试！");
    }
}
```

4、变成controller

MemberController.java

```java
@ResponseBody
@PostMapping("changeSessionStatus")
public Object changeSessionStatus(String sessionId, boolean status){
    Serializable id = TokenManager.getSession().getId();
    if(sessionId.equals(id)) return Result.error("不能自己踢出自己！");
    return sessionManager.changeSessionStatus(status, sessionId);
}
```

前端页面调用一下这个接口，这样就完成了踢出用户功能

#### 单用户登录

1、自定义一个shiro的filter

```java
public class KickoutFilter extends AccessControlFilter {


    static JedisManager jedisManager;
    static ShiroSessionRepository sessionRepository;

    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object o) 
        throws Exception {
        Subject subject = getSubject(request, response);
        // 非登录状态放行
        if(!subject.isAuthenticated() && !subject.isRemembered()) return true;
        // 获取当前sessionId
        Session session = subject.getSession();
        Serializable sessionId = session.getId();

        Boolean isKickOut = (Boolean) session.getAttribute(ShiroConstant.KICK_OUT_STATUS_KEY);
        if(null != isKickOut && isKickOut) return false;

        String id = String.valueOf(TokenManager.getUser().getId());
        byte[] keyByte = buildOnlineStatusKey(id).getBytes();
        byte[] sessionIdByte = jedisManager.get(ShiroConstant.DB_INDEX, keyByte);
        if (null != sessionIdByte) {
            // 如果存在记录，并且session一致则更新
            String sessionStr = new String(sessionIdByte);
            if(sessionId.equals(sessionStr)){
                jedisManager.setex(ShiroConstant.DB_INDEX, keyByte, sessionIdByte,
                                   ShiroConstant.SESSION_EXPIRE_TIME);
            }else{ // 如果 session 不一致，那就是在他处登录了
                Session oldSession = sessionRepository.getSession(sessionStr);
                jedisManager.setex(ShiroConstant.DB_INDEX, keyByte, sessionId.toString().getBytes(), 
                                   ShiroConstant.SESSION_EXPIRE_TIME);
                if(null != oldSession){ // 设置老session的踢出状态为true
                    oldSession.setAttribute(ShiroConstant.KICK_OUT_STATUS_KEY, true);
                    sessionRepository.saveSession(oldSession);
                    LoggerUtils.info(getClass(), "kickout old session success,oldId[%s]",
                                     sessionStr);
                }
            }
        }else{ // 如果不存在记录，则添加
            jedisManager.setex(ShiroConstant.DB_INDEX, keyByte, sessionId.toString().getBytes(), 
                               ShiroConstant.SESSION_EXPIRE_TIME);
        }
        return true;
    }

    @Override
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response) 
        throws Exception {
        Subject subject = getSubject(request, response);
        subject.logout();
        saveRequest(request);
        if(HTTPUtils.isAjax(request)) {
            Result result = Result.error("您已经被踢出，请重新登录或联系管理员！");
            HTTPUtils.out(response, result);
        }else{
            WebUtils.issueRedirect(request, response, ShiroConstant.KICK_OUT_URL);
        }
        return false;
    }

    private String buildOnlineStatusKey(String sessionId){
        return String.format("%s%s", ShiroConstant.ONLINE_STATUS_TAG, sessionId);
    }
    ... 省略set    
```

大致逻辑就是建立一个键值对来表示在线状态， online-status:[id]（id就是用户的id） 就是键，值就是当前访问的sessionId

首先刚登陆通过 jedisManager.get()查询有没有`online-status:[id]`这个标记的存在

+ 若存在 ， 则判断`online-status:[id]`标记存放的`sessionId`和当前访问的`sessionId`是否相同	
  + 若相同：则说明是同一个会话，更新一下TTL就行
  + 若不同：则说明不同会话，账号多地登录了，那么就更新这个标记为当前sessionId，并且将老的session添加一个踢出的属性

+ 不存在，则设置`online-status:[id]`的值为当前的`sessionId`

2、注册过滤器

在配置文件中将其注册，并应用

spring-shiro.xml

```xml
<!--自定义过滤器-->
<bean id="kickoutFilter" class="com.hemou.core.shiro.filter.KickoutFilter"/>

<!--静态注入-->
<bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
  <property name="staticMethod" value="com.hemou.core.shiro.filter.KickoutFilter.setJedisManager"/>
  <property name="arguments" ref="jedisManager"/>
</bean>
<bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
  <property name="staticMethod" value="com.hemou.core.shiro.filter.KickoutFilter.setSessionRepository"/>
  <property name="arguments" ref="jedisShiroSessionRepository"/>
</bean>
```

因为ShiroSessionRepository、JedisManager是静态变量，所以不能想往常使用@Autowired去注入，这里可以使用MethodInvokingFactoryBean来帮忙注入这两个静态变量

```xml
<!--shiro 过滤器配置-->
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    ...
    <property name="filters">
        <util:map>
            ...
            <entry key="kickout" value-ref="kickoutFilter"/>
			...
        </util:map>
    </property>
    <!-- 初始配置，现采用自定义  -->
    <property name="filterChainDefinitions" >
        <value>
            ...
            /user/** = user,online,kickout
            /member/** = user,roleOr["admin,user"],online,kickout
            /permission/** = user,roleOr["admin,perms"],online,kickout
            /role/** = user,roleOr["admin,role"],online,kickout
            /** = user
        </value>
    </property>
</bean>
```

配置完毕后当用户登录功能算是完成了

## 错误记录

### 修改个人信息

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

### PageInterceptor插件空指针异常

com.github.pagehelper.PageInterceptor插件空指针异常

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

### mybatis基本类型非空判断

mybatis 出现异常：There is no getter for property...

见下面参考

### 踢出用户

前端页面的所有空链，如下，尽量不要写成 href="#"，因为这样还会发送一次请求，导致shiro过滤器拦截，产生不预期的结果

```html
<a href="#">权限管理 <span class="caret"></span></a>
```

因改成如下

```html
<a href="javascript:">权限管理 <span class="caret"></span></a>
```

## 参考

[项目原始来源](https://www.sojson.com/shiro)

[No converter found for return value of type 异常问题](https://blog.csdn.net/qq_35917800/article/details/78452938) getter 和 setter

[shiro前端密码加密问题](https://www.bilibili.com/video/BV1YW411M7S3?p=11)

[ehcache配置文件说明](https://blog.csdn.net/u011955525/article/details/106970207/)

[freemarker相关](https://www.cnblogs.com/peijie-tech/articles/11831262.html)

[shiro-freemarker标签](https://www.sojson.com/blog/143.html)

[shiro过滤器工作原理](https://blog.csdn.net/achenyuan/article/details/78541529)

[shiro更改subject](https://blog.csdn.net/cpohui/article/details/106853932)

[Mybatis基本类型参数非空判断](https://www.cnblogs.com/zxxfz/p/11987980.html)

[静态环境下不允许类的参数是泛型类型](https://bbs.csdn.net/topics/390202559)

<div style="display:none">
    因为泛型是要在对象创建的时候才知道是什么类型的，而对象创建的代码执行先后顺序是static的部分，然后才是构造函数等等。所以在对象初始化之前static的部分已经执行了，如果你在静态部分引用的泛型，那么毫无疑问虚拟机根本不知道是什么东西，因为这个时候类还没有初始化。因此在静态方法、数据域或初始化语句中，为了类而引用泛型类型参数是非法的
</div>
[PO,BO,VO和POJO的区别](https://blog.csdn.net/u011870547/article/details/81077153)

[Could not resolve placeholder](https://blog.csdn.net/zhangjianming2018/article/details/86649025)

[getCanonicalName()含义](https://blog.csdn.net/zq1994520/article/details/78942684)

[shiro登录页报错](http://www.voidcn.com/article/p-fvbqbhse-e.html)

## 项目截图

<table>
    <tr>
        <td><img src="https://img2020.cnblogs.com/blog/1491349/202101/1491349-20210121180831587-1639958317.png"></td>
        <td><img src="https://img2020.cnblogs.com/blog/1491349/202101/1491349-20210121180837904-1545293861.png"></td>
    </tr>
    <tr>
        <td><img src="https://img2020.cnblogs.com/blog/1491349/202101/1491349-20210121180842637-1050045289.png"></td>
		<td><img src="https://img2020.cnblogs.com/blog/1491349/202101/1491349-20210121180848977-769761207.png"></td>
    </tr>
    <tr>
        <td><img src="https://img2020.cnblogs.com/blog/1491349/202101/1491349-20210121180853248-1610814750.png"></td>
		<td><img src="https://img2020.cnblogs.com/blog/1491349/202101/1491349-20210121181222048-2053619894.png"></td>
    </tr>
</table>
## 源码

https://github.com/HeMOua/shiro_authority