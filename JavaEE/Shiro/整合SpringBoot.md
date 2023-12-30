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