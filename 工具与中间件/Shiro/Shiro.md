Shiro 简介



Apache Shiro 是 Java 的一个安全框架。目前，使用 Apache Shiro 的人越来越多，因为它相当简单，对比 Spring Security，可能没有 Spring Security 做的功能强大，但是在实际工作时可能并不需要那么复杂的东西，所以使用小而简单的 Shiro 就足够了。对于它俩到底哪个好，这个不必纠结，能更简单的解决项目问题就好了。

Shiro 可以非常容易的开发出足够好的应用，其不仅可以用在 JavaSE 环境，也可以用在 JavaEE 环境。Shiro 可以帮助我们完成：认证、授权、加密、会话管理、与 Web 集成、缓存等。这不就是我们想要的嘛，而且 Shiro 的 API 也是非常简单；其基本功能点如下图所示：

![img](img/1.png)

**Authentication**：身份认证 / 登录，验证用户是不是拥有相应的身份；

**Authorization**：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限；

**Session Manager**：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通 JavaSE 环境的，也可以是如 Web 环境的；

**Cryptography**：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；

**Web Support**：Web 支持，可以非常容易的集成到 Web 环境；

**Caching**：缓存，比如用户登录后，其用户信息、拥有的角色 / 权限不必每次去查，这样可以提高效率；

**Concurrency**：shiro 支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去；

**Testing**：提供测试支持；

**Run As**：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；

**Remember Me**：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了。

**记住一点，Shiro 不会去维护用户、维护权限；这些需要我们自己去设计 / 提供；然后通过相应的接口注入给 Shiro 即可。**

接下来我们分别从外部和内部来看看 Shiro 的架构，对于一个好的框架，从外部来看应该具有非常简单易于使用的 API，且 API 契约明确；从内部来看的话，其应该有一个可扩展的架构，即非常容易插入用户自定义实现，因为任何框架都不能满足所有需求。

首先，我们从外部来看 Shiro 吧，即从应用程序角度的来观察如何使用 Shiro 完成工作。如下图：

![img](img/2.png)

可以看到：应用代码直接交互的对象是 Subject，也就是说 Shiro 的对外 API 核心就是 Subject；其每个 API 的含义：

**Subject**：主体，代表了当前 “用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是 Subject，如网络爬虫，机器人等；即一个抽象概念；所有 Subject 都绑定到 SecurityManager，与 Subject 的所有交互都会委托给 SecurityManager；可以把 Subject 认为是一个门面；SecurityManager 才是实际的执行者；

**SecurityManager**：安全管理器；即所有与安全有关的操作都会与 SecurityManager 交互；且它管理着所有 Subject；可以看出它是 Shiro 的核心，它负责与后边介绍的其他组件进行交互，如果学习过 SpringMVC，你可以把它看成 DispatcherServlet 前端控制器；

**Realm**：域，Shiro 从从 Realm 获取安全数据（如用户、角色、权限），就是说 SecurityManager 要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法；也需要从 Realm 得到用户相应的角色 / 权限进行验证用户是否能进行操作；可以把 Realm 看成 DataSource，即安全数据源。

也就是说对于我们而言，最简单的一个 Shiro 应用：

1. 应用代码通过 Subject 来进行认证和授权，而 Subject 又委托给 SecurityManager；
2. 我们需要给 Shiro 的 SecurityManager 注入 Realm，从而让 SecurityManager 能得到合法的用户及其权限进行判断。

**从以上也可以看出，Shiro 不提供维护用户 / 权限，而是通过 Realm 让开发人员自己注入。**

接下来我们来从 Shiro 内部来看下 Shiro 的架构，如下图所示：

![img](img/3.png)

**Subject**：主体，可以看到主体可以是任何可以与应用交互的 “用户”；

**SecurityManager**：相当于 SpringMVC 中的 DispatcherServlet 或者 Struts2 中的 FilterDispatcher；是 Shiro 的心脏；所有具体的交互都通过 SecurityManager 进行控制；它管理着所有 Subject、且负责进行认证和授权、及会话、缓存的管理。

**Authenticator**：认证器，负责主体认证的，这是一个扩展点，如果用户觉得 Shiro 默认的不好，可以自定义实现；其需要认证策略（Authentication Strategy），即什么情况下算用户认证通过了；

**Authrizer**：授权器，或者访问控制器，用来决定主体是否有权限进行相应的操作；即控制着用户能访问应用中的哪些功能；

**Realm**：可以有 1 个或多个 Realm，可以认为是安全实体数据源，即用于获取安全实体的；可以是 JDBC 实现，也可以是 LDAP 实现，或者内存实现等等；由用户提供；注意：Shiro 不知道你的用户 / 权限存储在哪及以何种格式存储；所以我们一般在应用中都需要实现自己的 Realm；

**SessionManager**：如果写过 Servlet 就应该知道 Session 的概念，Session 呢需要有人去管理它的生命周期，这个组件就是 SessionManager；而 Shiro 并不仅仅可以用在 Web 环境，也可以用在如普通的 JavaSE 环境、EJB 等环境；所有呢，Shiro 就抽象了一个自己的 Session 来管理主体与应用之间交互的数据；这样的话，比如我们在 Web 环境用，刚开始是一台 Web 服务器；接着又上了台 EJB 服务器；这时想把两台服务器的会话数据放到一个地方，这个时候就可以实现自己的分布式会话（如把数据放到 Memcached 服务器）；

**SessionDAO**：DAO 大家都用过，数据访问对象，用于会话的 CRUD，比如我们想把 Session 保存到数据库，那么可以实现自己的 SessionDAO，通过如 JDBC 写到数据库；比如想把 Session 放到 Memcached 中，可以实现自己的 Memcached SessionDAO；另外 SessionDAO 中可以使用 Cache 进行缓存，以提高性能；

**CacheManager**：缓存控制器，来管理如用户、角色、权限等的缓存的；因为这些数据基本上很少去改变，放到缓存中后可以提高访问的性能

**Cryptography**：密码模块，Shiro 提高了一些常见的加密组件用于如密码加密 / 解密的。

到此 Shiro 架构及其组件就认识完了，接下来挨着学习 Shiro 的组件吧。

# 入门程序

## 配置shiro环境

```java
// for this simple example quickstart, make the SecurityManager
// accessible as a JVM singleton.  Most applications wouldn't do this
// and instead rely on their container configuration or web.xml for
// webapps.  That is outside the scope of this simple quickstart, so
// we'll just do the bare minimum so you can continue to get a feel
// for things.
Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
SecurityManager securityManager = factory.getInstance();
SecurityUtils.setSecurityManager(securityManager);
```

这一段并不是主要的，因为大多数都是在容器或webapp的web.xml中配置

## 获取当前用户对象

```java
// get the currently executing user:
//获取当前用户对象
Subject currentUser = SecurityUtils.getSubject();

// Do some stuff with a Session (no need for a web or EJB container!!!)
Session session = currentUser.getSession();
session.setAttribute("someKey", "aValue");
String value = (String) session.getAttribute("someKey");
if (value.equals("aValue")) {
    log.info("Retrieved the correct value! [" + value + "]");
}
```

通过`SecurityUtils.getSubject()`获取当前用户对象，并通过当前用户获取`session`，`session`可以通过`setAttribute(‘key’， ‘value’)`或`getAttribute("someKey")`设置或读取属性

## 认证

```java
// let's login the current user so we can check against roles and permissions:
// 判断当前用户是否认证
if (!currentUser.isAuthenticated()) {
    // 通过账号密码获取当前用户的令牌，也就是token
    UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
    // 设置记住我
    token.setRememberMe(true);
    try {
        // 执行登录操作
        currentUser.login(token);
    } catch (UnknownAccountException uae) {
        log.info("There is no user with username of " + token.getPrincipal());
    } catch (IncorrectCredentialsException ice) {
        log.info("Password for account " + token.getPrincipal() + " was incorrect!");
    } catch (LockedAccountException lae) {
        log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                 "Please contact your administrator to unlock it.");
    }
    // ... catch more exceptions here (maybe custom ones specific to your application?
    catch (AuthenticationException ae) {
        //unexpected condition?  error?
    }
}
```

## 角色

```java
//test a role:
if (currentUser.hasRole("schwartz")) {
    log.info("May the Schwartz be with you!");
} else {
    log.info("Hello, mere mortal.");
}
```

## 权限

```java
//test a typed permission (not instance-level)
if (currentUser.isPermitted("lightsaber:wield")) {
    log.info("You may use a lightsaber ring.  Use it wisely.");
} else {
    log.info("Sorry, lightsaber rings are for schwartz masters only.");
}

//a (very powerful) Instance Level permission:
if (currentUser.isPermitted("winnebago:drive:eagle5")) {
    log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
             "Here are the keys - have fun!");
} else {
    log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
}
```

## 注销

```java
//all done - log out!
currentUser.logout();
```

# 整合Spring

## 引入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com</groupId>
  <artifactId>shiro-spring</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>shiro-spring Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <spring.version>4.3.25.RELEASE</spring.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>${spring.version}</version>
      <exclusions>
        <exclusion>
          <groupId>commons-logging</groupId>
          <artifactId>commons-logging</artifactId>
        </exclusion>
      </exclusions>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.9.5</version>
    </dependency>

    <!--slf4j-->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.30</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>1.7.30</version>
    </dependency>
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>
    <!--偷天换日包-->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jcl-over-slf4j</artifactId>
      <version>1.7.30</version>
    </dependency>

    <!-- json -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.7.3</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.7.3</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>2.7.3</version>
    </dependency>

    <dependency>
      <groupId>org.apache.shiro</groupId>
      <artifactId>shiro-all</artifactId>
      <version>1.4.0</version>
    </dependency>
  </dependencies>

  <build>
    <finalName>shiro-spring</finalName>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-war-plugin</artifactId>
          <version>3.2.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
      </plugins>
    </pluginManagement>

    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <includes>
          <include>**/*.properties</include>
          <include>**/*.xml</include>
          <include>**/*.tld</include>
        </includes>
        <filtering>false</filtering>
      </resource>
      <resource>
        <directory>src/main/java</directory>
        <includes>
          <include>**/*.properties</include>
          <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
      </resource>
    </resources>
  </build>
</project>
```

和Shiro有关的就一个

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-all</artifactId>
    <version>1.4.0</version>
</dependency>
```

## 配置

### web.xml

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

### SecurityManager

```xml
<!-- =========================================================
         Shiro Core Components - Not Spring Specific
         ========================================================= -->
<!-- Shiro's main business-tier object for web-enabled applications
         (use DefaultSecurityManager instead when there is no web environment)-->
<!--  
    1. 配置 SecurityManager!
    -->     
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="cacheManager" ref="cacheManager"/>
    <property name="authenticator" ref="authenticator"></property>

    <property name="realms">
        <list>
            <ref bean="jdbcRealm"/>
            <ref bean="secondRealm"/>
        </list>
    </property>

    <property name="rememberMeManager.cookie.maxAge" value="10"></property>
</bean>
```

### CacheManager

```xml
<!-- Let's use some enterprise caching support for better performance.  You can replace this with any enterprise
         caching framework implementation that you like (Terracotta+Ehcache, Coherence, GigaSpaces, etc -->
<!--  
    2. 配置 CacheManager. 
    2.1 需要加入 ehcache 的 jar 包及配置文件. 
    -->     
<bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
    <!-- Set a net.sf.ehcache.CacheManager instance here if you already have one.  If not, a new one
             will be creaed with a default config:
             <property name="cacheManager" ref="ehCacheManager"/> -->
    <!-- If you don't have a pre-built net.sf.ehcache.CacheManager instance to inject, but you want
             a specific Ehcache configuration to be used, specify that here.  If you don't, a default
             will be used.: -->
    <property name="cacheManagerConfigFile" value="classpath:ehcache.xml"/> 
</bean>
```

### Realm

```xml
<!-- Used by the SecurityManager to access security data (users, roles, etc).
         Many other realm implementations can be used too (PropertiesRealm,
         LdapRealm, etc. -->
<!-- 
     3. 配置 Realm 
     3.1 直接配置实现了 org.apache.shiro.realm.Realm 接口的 bean
    -->     
<bean id="jdbcRealm" class="com.atguigu.shiro.realms.ShiroRealm">
    <property name="credentialsMatcher">
        <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
            <property name="hashAlgorithmName" value="MD5"></property>
            <property name="hashIterations" value="1024"></property>
        </bean>
    </property>
</bean>
```

### lifecycleBeanPostProcessor

```xml
<!-- =========================================================
         Shiro Spring-specific integration
         ========================================================= -->
<!-- Post processor that automatically invokes init() and destroy() methods
         for Spring-configured Shiro objects so you don't have to
         1) specify an init-method and destroy-method attributes for every bean
            definition and
         2) even know which Shiro objects require these methods to be
            called. -->
<!--  
    4. 配置 LifecycleBeanPostProcessor. 可以自定的来调用配置在 Spring IOC 容器中 shiro bean 的生命周期方法. 
    -->       
<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>
```

### Shiro注解

```xml
<!-- Enable Shiro Annotations for Spring-configured beans.  Only run after
     the lifecycleBeanProcessor has run: -->
<!--  
5. 启用 IOC 容器中使用 shiro 的注解. 但必须在配置了 LifecycleBeanPostProcessor 之后才可以使用. 
-->     
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
      depends-on="lifecycleBeanPostProcessor"/>
<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
    <property name="securityManager" ref="securityManager"/>
</bean>
```
### shiroFilter

```xml
<!-- Define the Shiro Filter here (as a FactoryBean) instead of directly in web.xml -
         web.xml uses the DelegatingFilterProxy to access this bean.  This allows us
         to wire things with more control as well utilize nice Spring things such as
         PropertiesPlaceholderConfigurer and abstract beans or anything else we might need: -->
<!--  
    6. 配置 ShiroFilter. 
    6.1 id 必须和 web.xml 文件中配置的 DelegatingFilterProxy 的 <filter-name> 一致.
                      若不一致, 则会抛出: NoSuchBeanDefinitionException. 因为 Shiro 会来 IOC 容器中查找和 <filter-name> 名字对应的 filter bean.
    -->     
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="securityManager" ref="securityManager"/>
    <property name="loginUrl" value="/login.jsp"/>
    <property name="successUrl" value="/list.jsp"/>
    <property name="unauthorizedUrl" value="/unauthorized.jsp"/>

    <property name="filterChainDefinitionMap" ref="filterChainDefinitionMap"></property>

    <!--  
         配置哪些页面需要受保护. 
         以及访问这些页面需要的权限. 
         1). anon 可以被匿名访问
         2). authc 必须认证(即登录)后才可能访问的页面. 
         3). logout 登出.
         4). roles 角色过滤器
        -->
    <!--  
        <property name="filterChainDefinitions">
            <value>
                /login.jsp = anon
                /shiro/login = anon
                /shiro/logout = logout

                /user.jsp = roles[user]
                /admin.jsp = roles[admin]

                # everything else requires authentication:
                /** = authc
            </value>
        </property>
        -->
</bean>
```
#### 注意

`bean`的名字必须和web.xml中的`DelegatingFilterProxy`的filter名字一致，否则会报错

因为在`DelegatingFilterProxy`中有如下说明

```
Proxy for a standard Servlet Filter, delegating to a Spring-managed bean that
* implements the Filter interface. Supports a "targetBeanName" filter init-param
* in {@code web.xml}, specifying the name of the target bean in the Spring
* application context.
*
* <p>{@code web.xml} will usually contain a {@code DelegatingFilterProxy} definition,
* with the specified {@code filter-name} corresponding to a bean name in
* Spring's root application context. All calls to the filter proxy will then
* be delegated to that bean in the Spring context, which is required to implement
* the standard Servlet Filter interface.
```

## 授权URL匹配

![image-20200808194203291](img/image-20200808194203291.png)

![image-20200808194130738](img/image-20200808194130738.png)

![image-20200808194145993](img/image-20200808194145993.png)

## 认证

### 步骤

1. 获取当前的 Subject. 调用 SecurityUtils.getSubject();
2. 测试当前的用户是否已经被认证. 即是否已经登录. 调用 Subject 的 isAuthenticated() 
3. 若没有被认证, 则把用户名和密码封装为 UsernamePasswordToken 对象
1). 创建一个表单页面
2). 把请求提交到 SpringMVC 的 Handler
3). 获取用户名和密码. 
4. 执行登录: 调用 Subject 的 login(AuthenticationToken) 方法. 
5. 自定义 Realm 的方法, 从数据库中获取对应的记录, 返回给 Shiro.
1). 实际上需要继承 org.apache.shiro.realm.AuthenticatingRealm 类
2). 实现 doGetAuthenticationInfo(AuthenticationToken) 方法. 
6. 由 shiro 完成对密码的比对. 

**AccountController**

```java
@Controller
@RequestMapping("/shiro")
public class AccountController {

    @RequestMapping("/login")
    public String login(@RequestParam("username")String username
            , @RequestParam("password") String password){
        Subject currentUser = SecurityUtils.getSubject();
        if(!currentUser.isAuthenticated()){
            UsernamePasswordToken token = new UsernamePasswordToken(username, password);
            token.setRememberMe(true);
            try{
                currentUser.login(token);
                System.out.println("toke: 1." + token);
            }catch (AuthenticationException e){
                System.out.println("登录失败:" + e.getMessage());
            }
        }
        return "redirect:success";
    }
}
```

**ShiroRealm**

```java
public class ShiroRealm extends AuthenticatingRealm {

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //1. 把 AuthenticationToken 转换为 UsernamePasswordToken 
        UsernamePasswordToken upToken = (UsernamePasswordToken) token;

        //2. 从 UsernamePasswordToken 中来获取 username
        String username = upToken.getUsername();

        //3. 调用数据库的方法, 从数据库中查询 username 对应的用户记录
        System.out.println("从数据库中获取 username: " + username + " 所对应的用户信息.");

        //4. 若用户不存在, 则可以抛出 UnknownAccountException 异常
        if("unknown".equals(username)){
            throw new UnknownAccountException("用户不存在!");
        }

        //5. 根据用户信息的情况, 决定是否需要抛出其他的 AuthenticationException 异常. 
        if("monster".equals(username)){
            throw new LockedAccountException("用户被锁定");
        }

        //6. 根据用户的情况, 来构建 AuthenticationInfo 对象并返回. 通常使用的实现类为: SimpleAuthenticationInfo
        //以下信息是从数据库中获取的.
        //1). principal: 认证的实体信息. 可以是 username, 也可以是数据表对应的用户的实体类对象. 
        Object principal = username;
        //2). credentials: 密码. 
        Object credentials = null; //"fc1709d0a95a6be30bc5926fdb7f22f4";
        if("admin".equals(username)){
            credentials = "038bdaf98f2037b31f1e75b5b4c9b26e";
        }else if("user".equals(username)){
            credentials = "098d2c478e9c11555ce2823231e02ec1";
        }

        //3). realmName: 当前 realm 对象的 name. 调用父类的 getName() 方法即可
        String realmName = getName();
        //4). 盐值. 
        ByteSource credentialsSalt = ByteSource.Util.bytes(username);

        SimpleAuthenticationInfo info = null; //new SimpleAuthenticationInfo(principal, credentials, realmName);
        info = new SimpleAuthenticationInfo(principal, credentials, credentialsSalt, realmName);
        return info;
    }
}
```

### 盐值

```java
ByteSource credentialsSalt = ByteSource.Util.bytes(username);
```

## 密码比对

密码的比对:
通过 AuthenticatingRealm 的 credentialsMatcher 属性来进行的密码的比对!

## 替换加密方式

### 加密前端密码

1. 如何把前台传递的字符串加密为 MD5 
2. 替换当前 Realm 的 credentialsMatcher 属性. 直接使用 HashedCredentialsMatcher 对象, 并设置加密算法即可. 

```xml
<bean id="jdbcRealm" class="com.hemou.shiro.ShiroRealm">
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

### 加密明文密码

#### MD5加密

```java
public static void main(String[] args) {
    String hashAlgorithmName = "MD5";
    Object credentials = "123456";
    Object salt = ByteSource.Util.bytes("user");;
    int hashIterations = 1024;

    Object result = new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations);
    System.out.println(result);
}
```

#### SHA1加密

只需要改一下加密算法即可

```java
public static void main(String[] args) {
    String hashAlgorithmName = "MD5";
    Object credentials = "123456";
    Object salt = ByteSource.Util.bytes("user");;
    int hashIterations = 1024;

    Object result = new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations);
    System.out.println(result);
}
```



## 多Realm

1. 首先创建一个新的Realm

```java
public class SecondRealm extends AuthenticatingRealm {

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        System.out.println("[SecondRealm]");
```

2. 在springContext中配置新的Realm

```xml
<bean id="secondRealm" class="com.hemou.shiro.SecondRealm">
    <property name="credentialsMatcher">
        <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
            <!--加密算法-->
            <property name="hashAlgorithmName" value="SHA1"/>
            <!--加密次数-->
            <property name="hashIterations" value="1024"/>
        </bean>
    </property>
</bean>
```

3. 配置认证器ModularRealmAuthenticator

```xml
<bean class="org.apache.shiro.authc.pam.ModularRealmAuthenticator">
  <property name="realms">
    <list>
      <ref bean="jdbcRealm"/>
      <ref bean="secondRealm"/>
    </list>
  </property>
</bean>
```

4. 修改securityManager

添加`authenticator`属性，移除`realms`属性

```xml
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="cacheManager" ref="cacheManager"/>
    <property name="authenticator" ref="authenticator"/>
</bean>
```

==或者==修改第三步，不在`ModularRealmAuthenticator`中配置Realms，而是在securityManager中配置

```xml
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="cacheManager" ref="cacheManager"/>
    <property name="authenticator" ref="authenticator"/>
    <property name="realms">
        <list>
            <ref bean="jdbcRealm"/>
            <ref bean="secondRealm"/>
        </list>
    </property>
</bean>
```



## 认证策略

![image-20200808222235803](img/image-20200808222235803.png)

### 修改Authenticator认证器

```xml
<bean id="authenticator" class="org.apache.shiro.authc.pam.ModularRealmAuthenticator">
    <property name="realms">
        <list>
            <ref bean="jdbcRealm"/>
            <ref bean="secondRealm"/>
        </list>
    </property>
    <property name="authenticationStrategy">
        <bean class="org.apache.shiro.authc.pam.AllSuccessfulStrategy"/>
    </property>
</bean>
```

## 授权方式

![image-20200808223152598](img/image-20200808223152598.png)



## 拦截器

![image-20200808223220773](img/image-20200808223220773.png)

### 身份相关的过滤器

![image-20200808223325117](img/image-20200808223325117.png)

![image-20200808223409612](img/image-20200808223409612.png)

![image-20200808223944593](img/image-20200808223944593.png)

## 授权

首先继承`AuthorizingRealm`类，然后实现`doGetAuthorizationInfo`方法

```java
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
    System.out.println("认证中...");
    Set<String> roles = new HashSet<>();
    roles.add("user");
    Object principal = principals.getPrimaryPrincipal();
    if("admin".equals(principal)){
        roles.add("admin");
    }
    SimpleAuthorizationInfo info = new SimpleAuthorizationInfo(roles);
    return info;
}
```



## 标签

```html
<shiro:guest>
    游客访问 <a href = "login.jsp"></a>
</shiro:guest>
 
user 标签：用户已经通过认证\记住我 登录后显示响应的内容
<shiro:user>
    欢迎[<shiro:principal/>]登录 <a href = "logout">退出</a>
</shiro:user>
 
authenticated标签：用户身份验证通过，即 Subjec.login 登录成功 不是记住我登录的
<shiro:authenticted>
    用户[<shiro:principal/>] 已身份验证通过
</shiro:authenticted>
 
notAuthenticated标签：用户未进行身份验证，即没有调用Subject.login进行登录,包括"记住我"也属于未进行身份验证
<shiro:notAuthenticated>
    未身份验证(包括"记住我")
</shiro:notAuthenticated>
 
 
principal 标签：显示用户身份信息，默认调用
Subjec.getPrincipal()获取，即Primary Principal
<shiro:principal property = "username"/>
 
hasRole标签：如果当前Subject有角色将显示body体内的内容
<shiro:hashRole name = "admin">
    用户[<shiro:principal/>]拥有角色admin
</shiro:hashRole>
 
hasAnyRoles标签：如果Subject有任意一个角色(或的关系)将显示body体里的内容
<shiro:hasAnyRoles name = "admin,user">
    用户[<shiro:pricipal/>]拥有角色admin 或者 user
</shiro:hasAnyRoles>
 
lacksRole:如果当前 Subjec没有角色将显示body体内的内容
<shiro:lacksRole name = "admin">
    用户[<shiro:pricipal/>]没有角色admin
</shiro:lacksRole>
 
hashPermission:如果当前Subject有权限将显示body体内容
<shiro:hashPermission name = "user:create">
    用户[<shiro:pricipal/>] 拥有权限user:create
</shiro:hashPermission>
 
lacksPermission:如果当前Subject没有权限将显示body体内容
<shiro:lacksPermission name = "org:create">
    用户[<shiro:pricipal/>] 没有权限org:create
</shiro:lacksPermission>
```

## 注解

```java
@RequiresAuthenthentication:表示当前Subject已经通过login进行身份验证;即 Subjec.isAuthenticated()返回 true
 
@RequiresUser:表示当前Subject已经身份验证或者通过记住我登录的,
 
@RequiresGuest:表示当前Subject没有身份验证或者通过记住我登录过，即是游客身份
 
@RequiresRoles(value = {"admin","user"},logical = Logical.AND):表示当前Subject需要角色admin和user
 
@RequiresPermissions(value = {"user:delete","user:b"},logical = Logical.OR):表示当前Subject需要权限user:delete或者user:b
```

## 从数据库中初始化权限

如果权限设置过多，一个一个在xml会显得是非的麻烦，因此我们可以通过数据库获取这些信息。

因为`filterChainDefinitions`对应的是一个Map，所以我们通过spring的factory-bean就可以创建一个map

1. 首先创建Map工厂

```java
public class FilterChainDefinitionMapBuilder {

    public LinkedHashMap<String, String> buildFilter(){
        LinkedHashMap<String, String> map = new LinkedHashMap<>();

        map.put("/login.jsp", "anon");
        map.put("/shiro/login", "anon");
        map.put("/logout", "logout");
        map.put("/admin*", "roles[admin]");
        map.put("/user*", "roles[user]");
        map.put("/**", "authc");
        return map;
    }
}
```

2. 然后再把这个工厂交给spring管理

```xml
<bean id="chainDefinitionMapBuilder" class="com.hemou.shiro.FilterChainDefinitionMapBuilder"/>
```

3. 接着通过factory-bean创建这个Map

```xml
<bean id="chainDefinitionMap" factory-bean="chainDefinitionMapBuilder" factory-method="buildFilter"/>
```

4. 最后将这个map交给filter

```xml
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="securityManager" ref="securityManager"/>
    <property name="loginUrl" value="/login.jsp"/>
    <property name="successUrl" value="/success.jsp"/>
    <property name="unauthorizedUrl" value="/unauthorized.jsp"/>
    <!-- map -->
    <property name="filterChainDefinitionMap" ref="chainDefinitionMap"/>
</bean>
```

## 会话管理

Shiro 提供了完整的企业级会话管理功能，不依赖于底层容器（如web容器tomcat )，不管JavaSE还是JavaEE环境都可以使用，提供了会话管理、会话事件监听、会话存储/持久化、容器无关的集群、失效/过期支持、对Web的透明支持、SSO单点登录的支持等特性。

### 会话相关的 API

![image-20200809203718326](img/image-20200809203718326.png)

![image-20200809203729686](img/image-20200809203729686.png)

+ `Subject.getSession()`：即可获取会话；其等价于Subject.getSession(true)，即如果当前没有创建 Session 对象会创建一个；Subject.getSession(false)，如果当前没有创建 Session则返回null
+ `session.getId()`：获取当前会话的唯一标识
+ `session.getHost()`：获取当前Subject的主机地址
+ `session.getTimeout() & session.setTimeout(毫秒)`：获取/设置当前Session的过期时间
+ `session.getStartTimestamp() & session.getLastAccessTime()`：获取会话的启动时间及最后访问时间；如果是 JavaSE 应用需要自己定期调用 session.touch() 去更新最后访问时间；如果是 Web 应用，每次进入 ShiroFilter 都会自动调用 session.touch() 来更新最后访问时间。
+ `session.touch() & session.stop()`：更新会话最后访问时间及销毁会话；当Subject.logout()时会自动调用 stop 方法来销毁会话。如果在web中，调用 HttpSession. invalidate()也会自动调用Shiro Session.stop 方法进行销毁Shiro 的会话
+ `session.setAttribute(key, val) &session.getAttribute(key) &session.removeAttribute(key)`：设置/获取/删除会话属性；在整个会话范围内都可以对这些属性进行操作

### 会话验证

![image-20200809203639614](img/image-20200809203639614.png)

### 会话监听器

![image-20200809203817741](img/image-20200809203817741.png)

## SessionDao

![image-20200809202752899](img/image-20200809202752899.png)

+  AbstractSessionDAO 提供了 SessionDAO 的基础实现，如生成会话ID等
+ CachingSessionDAO 提供了对开发者透明的会话缓存的功能，需要设置相应的 CacheManager
+ MemorySessionDAO 直接在内存中进行会话维护
+ EnterpriseCacheSessionDAO 提供了缓存功能的会话维护，默认情况下使用 MapCache 实现，内部使用ConcurrentHashMap 保存缓存的会话。

### 配置

```xml
<!--Session ID 生成器-->
<bean id="sessionIdGenerator" class="org.apache.shiro.session.mgt.eis.JavaUuidSessionIdGenerator"/>

<!--SessionDao 继承 EnterpriseCacheSessionDAO-->
<bean id="sessionDao" class="com.hemou.shiro.MySessionDao">
    <property name="activeSessionsCacheName" value="shiro-activeSessionCache"/>
    <property name="sessionIdGenerator" ref="sessionIdGenerator"/>
</bean>

<!--会话管理器-->
<bean class="org.apache.shiro.session.mgt.DefaultSessionManager">
    <property name="globalSessionTimeout" value="1800000"/>
    <property name="deleteInvalidSessions" value="true"/>
    <property name="sessionValidationSchedulerEnabled" value="true"/>
    <property name="sessionDAO" ref="sessionDao"/>
</bean>
```

可以在ehcache.xml中配置其他信息，如下

```xml
<cache name="shiro-activeSessionCache"
       maxEntriesLocalHeap="2000"
       eternal="false"
       timeToIdleSeconds="3600"
       timeToLiveSeconds="0"
       overflowToDisk="false"
       statistics="true">
</cache>
```

### Session序列化

想要把Session存储到数据库中，首先要序列化

![image-20200809203450768](img/image-20200809203450768.png)

### 增删改查

![image-20200809203532409](img/image-20200809203532409.png)

![image-20200809203543997](img/image-20200809203543997.png)

## 缓存

### CacheManagerAware接口

![image-20200809203923915](img/image-20200809203923915.png)

### Realm缓存

![image-20200809203950927](img/image-20200809203950927.png)

### Session缓存

![image-20200809204019970](img/image-20200809204019970.png)

在本案例项目中，就是CacheManager的配置，如下

```xml
<bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
    <property name="cacheManagerConfigFile" value="classpath:ehcache.xml"/>
</bean>
```

如果我们将他去掉后，每次访问一个页面都要进行一次验证

而且我们也可以在Realm中单独的配置缓存管理器

![image-20200809204334098](img/image-20200809204334098.png)

## 记住我

### 概述



![image-20200809204428721](img/image-20200809204428721.png)

### 认证和记住我

![image-20200809204442926](img/image-20200809204442926.png)

![image-20200809204516230](img/image-20200809204516230.png)

![image-20200809204541074](img/image-20200809204541074.png)

### 实现

![image-20200809204616173](img/image-20200809204616173.png)

### 配置

![image-20200809204642927](img/image-20200809204642927.png)

```xml
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="cacheManager" ref="cacheManager"/>
    <property name="authenticator" ref="authenticator"/>
    <property name="realms">
        <list>
            <ref bean="jdbcRealm"/>
            <ref bean="secondRealm"/>
        </list>
    </property>
    <property name="rememberMeManager.cookie.maxAge" value="10"></property>
</bean>
```



# 整合springboot

## 引入依赖

```xml
<dependencies>

    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.4.0</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

## 配置

按照`Realm`、`SecurityManager`、`ShiroFilterFactoryBean`依次添加配置

### UserReaml

首先创建`UserRealm`对象并继承`AuthorizingRealm`

```java
@Slf4j
@Component
public class UserRealm extends AuthorizingRealm {

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        log.info("执行 doGetAuthorizationInfo 方法");
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        log.info("执行 doGetAuthenticationInfo 方法");
        return null;
    }
}
```

这样UserRealm算是配置完成

### DefaultWebSecurityManager

接着创建安全管理器

```java
// DefaultWebSecurityManager
@Bean
public DefaultWebSecurityManager defaultWebSecurityManager(UserRealm userRealm){
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();

    //关联userRealm
    securityManager.setRealm(userRealm);
    return securityManager;
}
```

### ShiroFilterFactoryBean

可以设置过滤规则，以及权限管理

```java
// ShiroFilterFactoryBean
@Bean
public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager securityManager){
    ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
    bean.setSecurityManager(securityManager);
    return bean;
}
```

#### shiro内置的过滤器

+ anon：无需认证就可以访问
+ authc：必须认证才能访问
+ user：必须要有记住我才能用
+ perms：拥有对某个资源的权限才能访问
+ role：拥有某个资源的角色权限才能才能访问

```java
@Bean
public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager securityManager){
    ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
    bean.setSecurityManager(securityManager);

    // 拦截页面
    Map<String, String> filterMap = new LinkedHashMap<>();
    filterMap.put("/user/*", "authc");
    
    // 设置/user/add页面需要user:add权限
    filterMap.put("/user/add", "perms[user:add]");
    bean.setFilterChainDefinitionMap(filterMap);

    // 设置登录页面
    bean.setLoginUrl("/login");
    return bean;
}
```

## 授权管理

首先在`ShiroConfig`里的`ShiroFilterFactoryBean`为某个界面添加权限设置

```java
filterMap.put("/user/add", "perms[user:add]");
```

然后在UserRealm的认证方法将用户数据传递到认证信息中，==并可以设置session==

```java
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
    log.info("执行 doGetAuthenticationInfo 方法");

    User user = User.builder().username("root").password("root").build();

    UsernamePasswordToken userToken = (UsernamePasswordToken) token;
    if (!userToken.getUsername().equals(user.getUsername())) {
        // 抛出异常 AccountException
        return null;
    }
	
    // 设置session
    Subject subject = SecurityUtils.getSubject();
    Session session = subject.getSession();
    session.setAttribute("loginUser", user);
    // 第一个参数就是用户的信息
    return new SimpleAuthenticationInfo(user, user.getPassword(), "");
}
```

然后再在授权验证方法中设置相应的权限

```java
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
    log.info("执行 doGetAuthorizationInfo 方法");
    SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
    Subject subject = SecurityUtils.getSubject();
    User currentUser = (User) subject.getPrincipal();
    info.addStringPermission(currentUser.getPerms());
    return info;
}
```

# 整合thymeleaf

## 引入依赖

```xml
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>
```

## ShiroDialect

```java
// 配置thymeleaf模板
@Bean
public ShiroDialect shiroDialect(){
    return new ShiroDialect();
}
```

## html约束

```html
<html xmlns:shiro="http://www.pollix.at/thymeleaf/shiro" xmlns:th="http://www.thymeleaf.org">
```

## 使用

```html
<div th:if="${session.loginUser==null}">
    <a th:href="@{/login}"></a>    
</div>

<div shiro:hasPermission="user:add">
    <a th:href="@{/user/add}">添加</a><br>
</div>

<div shiro:hasPermission="user:update">
    <a th:href="@{/user/update}">修改</a>
</div>
```



#  案例

