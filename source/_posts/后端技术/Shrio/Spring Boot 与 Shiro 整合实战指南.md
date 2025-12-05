---
title: Spring Boot 与 Shiro 整合实战指南
date: 2025-10-29 17:30:25
category: 后端
tags: Shiro
---

Spring Boot集成Shiro权限验证框架，可参考官方文档：

> https://shiro.apache.org/spring-boot.html

## 添加项目依赖

```xml

<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-web-starter</artifactId>
    <version>1.4.0</version>
</dependency>
```

Shiro配置类
ShiroConfig配置类：

```java

@ConfigurationProperties(prefix = "shiro")
@Configuration
public class ShiroConfig {

    @Autowired
    private ApplicationConfig applicationConfig;

    private List<String> pathDefinitions;

    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition chainDefinition = new
                DefaultShiroFilterChainDefinition();

        applicationConfig.getStaticDirs()
                .forEach(s -> chainDefinition.addPathDefinition(s, "anon"));
        this.getPathDefinitions().forEach(d -> {
            String[] defArr = d.split("=");
            chainDefinition
                    .addPathDefinition(StringUtils.trim(defArr[0]), StringUtils.trim(defArr[1]));
        });

        return chainDefinition;
    }

    @Bean
    public Realm systemRealm() {
        SystemRealm systemRealm = new SystemRealm();
        return systemRealm;
    }

    public List<String> getPathDefinitions() {
        return pathDefinitions;
    }

    public void setPathDefinitions(List<String> pathDefinitions) {
        this.pathDefinitions = pathDefinitions;
    }
}
```

ApplicationConfig：注入application.yml中的配置（此处略）；

SystemRealm实现类：

```java
public class SystemRealm extends AuthorizingRealm {

    @Autowired
    private SysAdminMapper sysAdminMapper;

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken)
            throws AuthenticationException {
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        token.setPassword(EcryptUtils.encode(String.valueOf(token.getPassword())).toCharArray
                ());

        SysAdminDO sysAdminParams = new SysAdminDO();
        sysAdminParams.setAdminLoginName(token.getUsername());
        SysAdminDO sysAdminDO = sysAdminMapper.selectByParams(sysAdminParams);

        AuthenticationInfo authInfo = null;
        if (sysAdminDO != null) {
            authInfo = new SimpleAuthenticationInfo(sysAdminDO, sysAdminDO.getAdminLoginPass(),
                    getName());
        }
        return authInfo;
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        /**
         * 以下为静态示例，实际应根据用户对应权限进行修改
         * 根据用户查询对应的角色和权限
         */
        SysAdminDO sysAdminDO = (SysAdminDO) super.getAvailablePrincipal(principalCollection);
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();

        Set<String> roles = new HashSet<>();
        roles.addAll(Arrays.asList("product", "operation"));
        authorizationInfo.setRoles(roles);

        Set<String> permissions = new HashSet<>();
        permissions.addAll(Arrays.asList("product:create", "product:del", "operation:update"));
        authorizationInfo.addStringPermissions(permissions);

        return authorizationInfo;
    }
}
```

应用配置文件
在application.yml中加入Shiro相关配置：

```
yaml
shiro:
  loginUrl: /login
  successUrl: /
  unauthorizedUrl: /error
  pathDefinitions:
    - /login/submit = anon
    - /logout = logout
    - /test = authc, roles[product], perms[operation:update]
    - /** = authc
loginUrl：未认证用户重定向的登录页面地址；

successUrl：认证成功后跳转的页面；

unauthorizedUrl：认证失败后跳转的页面；

pathDefinitions：定义路径授权规则；
```

更多配置参数请参考官方文档：

https://shiro.apache.org/spring-boot.html#configuration-properties

登录服务实现

```java

@Override
public SysAdminDO login(LoginForm form) {
    UsernamePasswordToken token = new UsernamePasswordToken(form.getLoginName(),
            form.getLoginPassword());
    token.setRememberMe(true);
    Subject currentUser = getSubject();
    try {
        currentUser.login(token);
    } catch (Exception e) {
        logger.error("登录验证失败：", e);
    }
    return (SysAdminDO) currentUser.getPrincipal();
}
```

内置过滤器说明
anon、authc等过滤器的详细定义参考类：

```java
org.apache.shiro.web.filter.mgt.DefaultFilter
```

官方文档定义：

http://shiro.apache.org/web.html#default-filters