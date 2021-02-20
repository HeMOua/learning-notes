# SpringSecurity认证流程

## SpringSecurity配置

SecurityConfig.java

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    // CRSF禁用，不使用session
    http.csrf().disable();
    // 基于token，所以不需要session
    http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    // 过滤请求
    http.authorizeRequests()
        .antMatchers("/login", "/captchaImage").anonymous()
        .anyRequest().authenticated();
    ....
    // 添加JWT filter
    http.addFilterBefore(authenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
}
```

从上面可以看到我们设置了放行 `/login`和 `/captchaImage` 请求，而其他请求就需要通过认证才可访问。下面正式分析登录认证流程

## 登录认证

首先分析一下流程

1、前端填写完表单数据后，发送请求 `[post] /login`，传递 username、password、code、uuid这四个数据

2、后端收到 `[post] /login`请求，来到相应的 controller 处理方法

```java
@RestController
public class SysLoginController {

    @Autowired
    private SysLoginService loginService;

    @PostMapping("login")
    public Result login(String username, String password, String code, String uuid){
        Result result = Result.success();
        String token = loginService.login(username, password, code, uuid);
        result.put(Constants.TOKEN, token);
        return result;
    }
```

在这里调用 loginService 的 login方法，其中内部是具体的验证登录逻辑，验证通过后返回一个token，然后回传给前端。前端这时就可以将token数据存入本地了

3、接下来进入 `loginService.login()`内，进行详细分析

```java
@Component
public class SysLoginService {
    @Autowired private RedisCache redisCache;
    @Autowired private TokenService tokenService;
    @Autowired private AuthenticationManager authenticationManager;
    
    public String login(String username, String password, String code, String uuid) {
        String verifyKey = Constants.CAPTCHA_CODE_TAG + uuid;
        String verifyCode = redisCache.getCacheObject(verifyKey);
        ...省去验证码校验逻辑，校验失败会抛出异常
            
        Authentication authenticate = null;
        try {
        // 认证 该方法会去调用UserDetailsServiceImpl.loadUserByUsername
            authenticate = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(username, password));
        } catch (Exception e) {
            if (e instanceof BadCredentialsException) {
                throw new UserPasswordNotMatchException();
            } else {
                throw new CustomException(e.getMessage());
            }
        }
        return tokenService.createToken((LoginUser) authenticate.getPrincipal());
    }
}
```

3.1 首先进行验证码校验逻辑，不通过时会抛出异常

3.2 通过 `AuthenticationManager` 的`authenticate()`获取认证信息

下面来详细分析一下 `authenticationManager.authenticate()`的**认证过程**

在**分析之前**先来了解一下 `UsernamePasswordAuthenticationToken` 这个类，它拥有两个构造方法

```java
// 1、只有两个参数的构造方法表示[当前没有认证]
public UsernamePasswordAuthenticationToken(Object principal, Object credentials)
// 2、拥有三个参数的构造方法表示[当前已经认证完毕]
public UsernamePasswordAuthenticationToken(Object principal, Object credentials, Collection<? extends GrantedAuthority> authorities)
```

![image-20210211230717227](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111209270-25593274.png)

下面**开始分析认证过程**

![png](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111214429-1207745914.png)



1）实现了`AuthenticationMananger`的`ProviderManger`调用接口的`authenticate`方法

![image-20210211221629377](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111218902-477924771.png)

 2）然后遍历所有的 AuthenticProvider，其中的`supports`方法，返回时一个boolean值，参数是一个Class，就是根据Token的类来确定用什么Provider来处理

而源码中的toTest类，就是我们认证传递的 `UsernamePasswordAuthenticationToken`，而他对应的provider就是`AbstractUserDetailsAuthenticationProvider`

AbstractUserDetailsAuthenticationProvider.java

```java
@Override
public boolean supports(Class<?> authentication) {
    return (UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication));
}
```

3）找到合适的Provider后，在本例中也即是AbstractUserDetailsAuthenticationProvider（抽象类），会调用provider 的authenticate 方法

![image-20210211224205852](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111224769-231714021.png)

4）从下面可以看到  retrieveUser 方法返回一个 UserDetails

![image-20210211224415630](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111229541-672939290.png)

5）接着深入，可以发现 DaoAuthenticationProvider 继承了 AbstractUserDetailsAuthenticationProvider，所以DaoAuthenticationProvider 才是真正的实现类，他会调用 retrieveUser  方法，接着调用 loaderUserByUsername() 方法

![image-20210211224719481](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111232769-661828572.png)

看到 loaderUserByUsername()，应该就很熟悉了，因为这就是我们自己实现 `UserDetailsService` 接口，自定义的认证过程

6）接着看我们自定义的 UserDetailServiceImpl 

```java
@Service("userDetailsService")
public class UserDetailServiceImpl implements UserDetailsService {
    ...
    @Autowired private ISysUserService userService;
    @Autowired private SysPermissionService permissionService;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        SysUser user = userService.selectUserByUserName(username);
        if (null == user) {
            log.info("登录用户：{} 不存在.", username);
            throw new UsernameNotFoundException("登录用户：" + username + " 不存在");
        } else if (UserStatus.DELETED.getCode().equals(user.getDelFlag())) {
            log.info("登录用户：{} 已被删除.", username);
            throw new BaseException("对不起，您的账号：" + username + " 已被删除");
        } else if (UserStatus.DISABLE.getCode().equals(user.getStatus())) {
            log.info("登录用户：{} 已被停用.", username);
            throw new BaseException("对不起，您的账号：" + username + " 已停用");
        }
        return createLoginUser(user);
    }

    public UserDetails createLoginUser(SysUser user) {
        return new LoginUser(user, permissionService.getMenuPermission(user));
    }
}
```

上面就是我们自己的认证逻辑。通过一个唯一标识查询用户，在这里就是username，当所有校验都通过后就会调用 createLoginUser 方法，装填用户拥有的权限以及**从数据库中获取的密码**，返回一个 LoginUser 对象，而这个对象实现了 UserDetails接口。（创建token的方法以及LoginUser.java可以参考文末的相关代码）

7）然后我们回看AbstractUserDetailsAuthenticationProvider 的 authenticate 方法

![image-20210211230406093](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111242458-1187130244.png)

8）接着深入，可以发现`createSuccessAuthentication`方法创建了一个UsernamePasswordAuthenticationToken，并且他的构造方法有三个参数，这表明这个token是已近认证过后的

![image-20210211230531473](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111246874-1801079482.png)

4、至此认证已经结束，我再回到 `loginService.login()`这个我们自己写的方法内，上面分析的8个步骤，也就是调用 `authenticationManager.authenticate()`的过程会返回一个 Authentication，然后就可以利用这个 Authentication 生成一个 token。

loginService.java

```java
return tokenService.createToken((LoginUser) authenticate.getPrincipal());
```

5、接着回到前面第二步controller调用的 `loginService.login()`，这时它已近拿到了 token ，于是将其返回到前端。前端收到相应后，就可以把这个token存在本地，以后每次访问请求时都带上这个token 信息。

```java
@PostMapping("login")
public AjaxResult login(String username, String password, String code, String uuid){
    AjaxResult result = AjaxResult.success();
    String token = loginService.login(username, password, code, uuid);
    result.put(Constants.TOKEN, token);
    return result;
}
```

## 请求认证

1、上面说到前端每次发送请求都带上这个 token，但是为啥带上这个 token，SpringSecurity就会认为此次请求已近认证通过了呢，别忘了，因为之前配置 SecurityConfig，如下

```java
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        ...
        // 过滤请求
        http.authorizeRequests()
            .antMatchers("/login", "/captchaImage").anonymous()
            .anyRequest().authenticated();
        // 添加JWT filter,后面马上就讲
        http.addFilterBefore(authenticationTokenFilter, 
                             UsernamePasswordAuthenticationFilter.class);
    }
```
因为我们设置了放行 `/login`请求，所以才没遭受拦截，而其他请求都是要被拦截的

2、因此我们就要自定义一个JWT filter，使用 `addFilterBefore` 把它添加到过滤器列表中，下面来看我们写的 JWT过滤器

JwtAuthenticationTokenFilter.java

```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Autowired
    private TokenService tokenService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
        // 1)
        LoginUser loginUser = tokenService.getLoginUser(request);
        // 2)
        if (ObjectUtil.isNotNull(loginUser) && ObjectUtil.isNull(SecurityUtils.getAuthentication()))
        {
            // 3.1)
            tokenService.verifyToken(loginUser);
            // 3.2)
            UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(loginUser, null, loginUser.getAuthorities());
            authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
            SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        }
        chain.doFilter(request, response);
    }
}
```

1）TokenService#getLoginUser() 是我们编写的一个方法，可以从 request 中获取 token，然后再解析成LoginUser，也就是我们之前编写继承了UserDetails的类

2）如果1）能解析成功，也就是loginUser不为空，就说明请求含带了token认证信息，又因为这是个前后端分离项目，`SecurityUtils.getAuthentication()`肯定获取不到当前请求的认证信息

```java
public static Authentication getAuthentication() {
    return SecurityContextHolder.getContext().getAuthentication();
}
```

3）而没有认证信息，这次请求势必会被拦截下来，所以我们要手动加上这个认证信息

3.1）我们先刷新一下token，也就是在redis缓存中更新一下到期时间（刷新token的方法可以参考文末的相关代码）

3.2）然后创建UsernamePasswordAuthenticationToken，注意这是个有三个 参数的构造方法，前面也说了，这代表已经经过认证，然后 `SecurityContextHolder.getContext().setAuthentication();`设置一下认证信息

这样每次带token的请求，都会有了认证信息，也就不会被拦截了

3、

> 但是为了更深刻的了解，我们接下来具体分析一下流程。

再次之前先介绍一个类 `FilterSecurityInterceptor`：是一个方法级的权限过滤器，基本位于过滤链的最底部，下面来看看源码。

![image-20210212102053438](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111253863-952441950.png)

下面来打个断点，查看一下

![image-20210212102439157](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111257853-1567880337.png)

这说明来到`beforeInvocation`方法时我们前面编写的jwtFilter已经被执行，认证信息已近被手动添加过了

![image-20210212102758021](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111301333-1915938085.png)

进入`beforeInvocation()`里面，由调试信息可以看到当前请求需要被认证

![image-20210212103144116](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111304839-493673989.png)

接着我们进入`authenticateIfRequired`方法的内部

![image-20210212103331881](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111309890-2103762999.png)

因为我们之前jwtfilter手动添加了认证信息，所以`authenticateIfRequired`就直接返回了authentication，

否则的还要进行`authenticationManager.authenticate();`进行验证，如果没有之前jwtfilter手动添加认证信息，那么中途一定会抛出异常，导致此次请求失败被拦截

至此也没啥好讲了，filterInvocation.getChain().doFilter() 调用我们的后台服务了

![image-20210212104009332](https://img2020.cnblogs.com/blog/1491349/202102/1491349-20210212111313880-319009086.png)

## 相关代码

### TokenService.java

```java
@Component
public class TokenService {

    protected static final long MILLIS_SECOND = 1000;
    
    protected static final long MILLIS_MINUTE = 60 * MILLIS_SECOND;

    private static final long MILLIS_MINUTE_20 = 20 * 60 * 1000L;

    // 令牌自定义标识
    @Value("${token.header}")
    private String header;

    // 令牌秘钥
    @Value("${token.secret}")
    private String secret;

    // 令牌有效期（默认30分钟）
    @Value("${token.expireTime}")
    private int expireTime;

    @Autowired
    private RedisCache redisCache;

    /**
     * 创建令牌
     *
     * @param loginUser 用户信息
     * @return 令牌
     */
    public String createToken(LoginUser loginUser) {
        String token = IdUtil.fastUUID();
        loginUser.setToken(token);
        refreshToken(loginUser);

        Map<String, Object> claims = new HashMap<>();
        claims.put(Constants.LOGIN_TOKEN_KEY, token);
        return createToken(claims);
    }

    /**
     * 验证令牌有效期，相差不足20分钟，自动刷新缓存
     *
     * @param loginUser 登录用户
     * @return 令牌
     */
    public void verifyToken(LoginUser loginUser) {
        long expireTime = loginUser.getExpireTime();
        long currentTime = System.currentTimeMillis();
        if (expireTime - currentTime <= MILLIS_MINUTE_20) {
            refreshToken(loginUser);
        }
    }

    /**
     * 刷新令牌有效期
     *
     * @param loginUser 登录信息
     */
    public void refreshToken(LoginUser loginUser) {
        loginUser.setLoginTime(System.currentTimeMillis());
        loginUser.setExpireTime(loginUser.getLoginTime() + expireTime * MILLIS_MINUTE);
        // 根据uuid将loginUser缓存
        String userKey = getTokenKey(loginUser.getToken());
        redisCache.setCacheObject(userKey, loginUser, expireTime, TimeUnit.MINUTES);
    }

    /**
     * 获取 token 的 redis 键前缀
     *
     * @param uuid
     * @return
     */
    private String getTokenKey(String uuid) {
        return Constants.LOGIN_TOKEN_TAG + uuid;
    }

    /**
     * 从数据声明生成令牌
     *
     * @param claims 数据声明
     * @return 令牌
     */
    private String createToken(Map<String, Object> claims) {
        return Jwts.builder().setClaims(claims).signWith(SignatureAlgorithm.HS512, secret).compact();
    }

    /**
     * 从令牌中获取数据声明
     *
     * @param token 令牌
     * @return 数据声明
     */
    private Claims parseToken(String token) {
        return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
    }

    /**
     * 获取用户身份信息
     *
     * @return 用户信息
     */
    public LoginUser getLoginUser(HttpServletRequest request) {
        // 获取请求携带的令牌
        String token = getRespToken(request);
        if (StringUtils.isNotEmpty(token)) {
            Claims claims = parseToken(token);
            // 解析对应的权限以及用户信息
            String uuid = (String) claims.get(Constants.LOGIN_TOKEN_KEY);
            String userKey = getTokenKey(uuid);
            return redisCache.getCacheObject(userKey);
        }
        return null;
    }

    /**
     * 获取请求token
     *
     * @param request
     * @return token
     */
    private String getRespToken(HttpServletRequest request) {
        String token = request.getHeader(this.header);
        if (StringUtils.isNotEmpty(token) && token.startsWith(Constants.REQ_TOKEN_PREFIX)) {
            token = token.replace(Constants.REQ_TOKEN_PREFIX, "");
        }
        return token;
    }
}
```

### LoginUser.java

```java
public class LoginUser implements UserDetails {
    private static final long serialVersionUID = 1821121071052157802L;

    /** 用户唯一标识 */
    private String token;

    /** 登陆时间 */
    private Long loginTime;

    /** 过期时间 */
    private Long expireTime;
    ...
    /** 权限列表 */
    private Set<String> permissions;

    /** 用户信息 */
    private SysUser user;
    ...
    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUserName();
    }
    ...
```

## 参考

https://www.cnblogs.com/ymstars/p/10626786.html

https://www.jianshu.com/p/d5ce890c67f7

https://gitee.com/y_project/RuoYi-Vue