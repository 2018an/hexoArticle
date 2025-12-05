---
title: Shiro Realm 权限验证流程与缓存机制详解
date: 2025-10-29 17:30:25
category: 后端
tags: Shiro
---

我们可以定义多个继承AuthenticatingRealm的Realm权限类。

在这种情况下，Shiro的验证策略和执行顺序是怎样的呢？

## 验证策略

通过分析源码，Shiro的Spring Boot自动配置采用"至少一个通过"策略，即只要有一个权限类验证通过就判定为有权限。

自动配置类：

> org.apache.shiro.spring.config.web.autoconfigure.ShiroWebAutoConfiguration

```java

@Bean
@ConditionalOnMissingBean
@Override
protected AuthenticationStrategy authenticationStrategy() {
    return super.authenticationStrategy();
}
```

```java
protected AuthenticationStrategy authenticationStrategy() {
    return new AtLeastOneSuccessfulStrategy();
}
```

Shiro还支持全部通过、首个通过等其他策略，更多信息可查看Shiro包中的权限策略类。

org.apache.shiro.authc.pam

执行顺序
Shiro按照Spring Boot配置类中定义Realm Bean的顺序进行权限验证。

完整验证流程
假设现有R1、R2两个权限类，某个方法或路径配置了A角色和B、C权限。Shiro会在R1中查找A角色，找到则继续验证其他权限，找不到则根据策略决定下一步操作。如果不是要求全部通过的策略，会继续在R2中查找A角色，如果仍然找不到则跳转到指定的未授权页面。B、C权限的验证流程与此一致。

Shiro缓存机制
为提高权限验证效率，Shiro对认证和授权过程提供了缓存控制开关。

需要了解的权限类层次结构是：每个Realm都继承AuthorizingRealm，而AuthorizingRealm又继承自AuthenticatingRealm。AuthenticatingRealm处理认证逻辑，AuthorizingRealm处理授权逻辑。

通过分析AuthorizingRealm和AuthenticatingRealm源码，默认情况下认证缓存是关闭的，而授权缓存是开启的。

authorizationCachingEnabled = true; // 授权缓存

authenticationCachingEnabled = false; // 认证缓存

仅开启默认缓存还不够，还需要设置CacheManager，如下所示：

```java

@Bean
public CacheManager cacheManager() {
    return new MemoryConstrainedCacheManager();
}
```

具体缓存逻辑可参考以下源码：

org.apache.shiro.realm.AuthorizingRealm#getAuthorizationInfo

org.apache.shiro.realm.AuthenticatingRealm#getAuthenticationInfo

缓存开关配置
在某些场景下，权限可能需要动态调整，因此希望某些Realm不使用缓存。

我们可以在特定Realm中手动关闭其授权缓存：

```java
this.setAuthorizationCachingEnabled(false);
```

同样，默认关闭的认证缓存也可以通过设置启用。

根据具体业务需求灵活调整缓存配置。