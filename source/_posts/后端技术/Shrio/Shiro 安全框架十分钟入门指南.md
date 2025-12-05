---
title: Shiro 安全框架十分钟入门指南
date: 2025-10-29 17:30:25
category: 后端
tags: Shiro
---

## 当前用户识别

现在我们可以开始探索安全框架的核心功能——执行安全验证操作。

在应用程序安全防护场景中，最常关注的问题是"当前操作用户身份"或"当前用户是否具备执行特定操作的权限"。

在编码开发或界面设计过程中，这些问题频繁出现：应用程序通常基于用户上下文构建，需要根据每个用户的权限范围展示相应功能。因此，基于当前用户进行安全控制是最自然的方式。

Shiro框架通过Subject概念来抽象表示"当前用户"。

在绝大多数运行环境中，可以通过以下方式获取当前执行用户：

```java
Subject currentUser = SecurityUtils.getSubject();
```

通过SecurityUtils.getSubject()方法，我们能够获取当前执行的Subject实例。Subject是一个安全术语，本质上代表"
当前执行用户的安全视图"。之所以不直接命名为"User"，是因为"User"通常特指人类用户。

在安全领域，"Subject"不仅可以表示人类用户，还可以表示第三方进程、定时任务、守护进程账户等任何与软件交互的实体。

对于大多数应用场景，您可以将Subject理解为Shiro框架中的"用户"概念。

在独立应用程序中调用getSubject()会返回基于特定位置用户数据的Subject，而在服务器环境（如Web应用）中，它会获取基于当前线程或传入请求关联用户数据的Subject。

用户会话管理
获得Subject实例后，我们可以进行哪些操作？
如果需要在应用程序的当前会话中存储用户相关数据，可以获取其会话：

```java
Session session = currentUser.getSession();
session.setAttribute("someKey","aValue");
```

Session是Shiro特有的会话实例，提供了与HttpSession相似的功能，但具备额外优势且有一个重要区别：它不依赖HTTP环境！

在Web应用程序中，默认Session基于HttpSession实现。但在非Web环境（如本教程的简单应用）中，Shiro会自动启用其企业级会话管理功能。这意味着无论部署环境如何，您都可以在任何应用层使用相同的会话API。这为应用程序开发开辟了新可能，因为需要会话功能的应用程序不再强制依赖HttpSession或有状态会话Bean。同时，各种客户端技术现在能够共享会话数据。

权限验证机制
权限检查只能针对已知身份的用户进行。我们获取的Subject实例代表当前用户，但当前用户具体是谁？在用户完成登录前，他们处于匿名状态。因此，我们需要进行登录验证：

```java
if(!currentUser.isAuthenticated()){
//通过图形界面收集用户凭证信息
//如用户名/密码表单、X509证书、OpenID等
//此处使用最常见的用户名/密码示例
UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
//支持"记住我"功能（无需额外配置，内置支持）
    token.setRememberMe(true);
    currentUser.login(token);
}
```

实现非常简单！

但如果登录尝试失败如何处理？您可以捕获各种具体异常来了解失败原因，并作出相应处理：

```java
try{
        currentUser.login(token);
//无异常抛出表示登录成功
}catch(
UnknownAccountException uae){
        //用户名不存在系统，显示错误提示
        }catch(
IncorrectCredentialsException ice){
        //密码不匹配，请重试
        }catch(
LockedAccountException lae){
        //账户被锁定，显示提示信息
        }
        ...可检查其他异常类型...
        }catch(
AuthenticationException ae){
        //意外错误条件处理
        }
```

您可以检查多种异常类型，或根据需求抛出Shiro未提供的自定义异常。请参考AuthenticationException JavaDoc获取详细信息。

实用提示

最安全的做法是向用户显示通用的登录失败信息，避免为潜在攻击者提供系统相关信息。

用户登录成功后，我们还能进行哪些操作？

例如，获取用户身份信息：

```java
//输出用户标识主体（本例中为用户名）
log.info("用户 ["+currentUser.getPrincipal() +"] 登录成功");
```

检查用户是否拥有特定角色：

```java
if(currentUser.hasRole("schwartz")){
        log.

info("愿原力与您同在！");
}else{
        log.

info("您好，普通用户");
}
```

验证用户是否具备对特定类型实体的操作权限：

```java
if(currentUser.isPermitted("lightsaber:weild")){
        log.

info("您可以使用光剑戒指，请谨慎使用");
}else{
        log.

info("抱歉，光剑戒指仅限原力大师使用");
}
```

当然，我们还可以执行更精细的实例级权限检查——判断用户是否有权访问特定类型的某个具体实例：

```java
if(currentUser.isPermitted("winnebago:drive:eagle5")){
        log.

info("您有权驾驶牌照为'eagle5'的Winnebago，这是钥匙-祝您愉快！");
}else{
        log.

info("抱歉，您无权驾驶'eagle5' Winnebago");
}
```

非常简单，对吧？

最后，当用户结束应用程序使用时，可以执行注销操作：

```java
currentUser.logout(); //移除所有身份信息并使其会话失效
```

内容总结
希望本教程能帮助您了解如何在基础应用程序中配置Shiro，并理解Shiro的核心设计理念。