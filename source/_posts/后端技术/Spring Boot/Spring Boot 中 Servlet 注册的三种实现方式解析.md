---
title: Spring Boot 中 Servlet 注册的三种实现方式解析
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

## Spring Boot中集成传统Servlet组件的三种方案

在现代化Spring Boot应用中，有时仍需要集成传统的Servlet组件。本文将详细介绍三种不同的注册方式，帮助你在Spring
Boot框架中灵活配置Servlet、Filter和Listener。

## 方案一：通过Spring Bean注册

Spring Boot提供了专用的Bean封装类来简化传统Web组件的注册：

- `ServletRegistrationBean` - 用于注册Servlet
- `FilterRegistrationBean` - 用于注册Filter
- `ServletListenerRegistrationBean` - 用于注册Listener

以下是一个Servlet注册的完整示例：

```java
// 自定义Servlet实现
public class CustomServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp)
            throws IOException {
        String name = getServletConfig().getInitParameter("name");
        String gender = getServletConfig().getInitParameter("gender");

        resp.getOutputStream().println("用户名: " + name);
        resp.getOutputStream().println("性别: " + gender);
    }
}

// 在Spring配置类中注册
@Bean
public ServletRegistrationBean<CustomServlet> customServletRegistration() {
    ServletRegistrationBean<CustomServlet> registration =
            new ServletRegistrationBean<>(new CustomServlet(), "/api/custom");
    registration.addInitParameter("name", "springboot");
    registration.addInitParameter("gender", "male");
    return registration;
}
```

## 方案二：使用注解声明并扫描

自Servlet 3.0规范起，传统的`web.xml`配置文件不再是强制要求。通过以下注解可以直接声明Web组件：

- `@WebServlet` - 替代`<servlet>`配置
- `@WebFilter` - 替代`<filter>`配置
- `@WebListener` - 替代`<listener>`配置

如下图所示，Servlet 3.1规范明确支持这些注解：
![](img/18-8-28-48900950.jpg)

### 注解配置Servlet示例

```java

@WebServlet(
        name = "apiServlet",
        urlPatterns = "/api/data",
        asyncSupported = true,
        initParams = {
                @WebInitParam(name = "appName", value = "DemoApp"),
                @WebInitParam(name = "version", value = "1.0")
        }
)
public class ApiServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp)
            throws IOException {
        String appName = getServletConfig().getInitParameter("appName");
        String version = getServletConfig().getInitParameter("version");

        resp.setContentType("text/plain");
        resp.getWriter().write("应用: " + appName + ", 版本: " + version);
    }
}
```

### 注解配置Filter示例

```java

@WebFilter(
        filterName = "securityFilter",
        urlPatterns = "/secure/*",
        initParams = {
                @WebInitParam(name = "tokenHeader", value = "Authorization"),
                @WebInitParam(name = "timeout", value = "3600")
        }
)
public class SecurityFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) {
        System.out.println("安全过滤器初始化完成");
        String header = filterConfig.getInitParameter("tokenHeader");
        System.out.println("令牌头字段: " + header);
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {
        System.out.println("执行安全验证逻辑");
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {
        System.out.println("安全过滤器销毁");
    }
}
```

**重要提示**：当使用Spring Boot内嵌服务器（如Tomcat）时，需要在配置类上添加`@ServletComponentScan`
注解来启用组件扫描功能。如果部署到独立的外部服务器，则无需此注解。

## 方案三：动态编程式注册

对于需要动态控制组件注册的场景，可以实现`ServletContextInitializer`接口，并在Spring中将其声明为Bean。

`ServletContext`提供了丰富的动态注册方法：
![](img/18-8-28-86108280.jpg)

动态注册Servlet的示例：

```java

@Component
public class DynamicServletRegistrar implements ServletContextInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {
        // 动态注册Servlet
        ServletRegistration.Dynamic dynamicServlet = servletContext
                .addServlet("dynamicServlet", DynamicServlet.class);
        dynamicServlet.addMapping("/dynamic/*");
        dynamicServlet.setInitParameter("environment", "production");
        dynamicServlet.setInitParameter("maxConnections", "100");
        dynamicServlet.setLoadOnStartup(1);

        // 同样可以动态注册Filter和Listener
        FilterRegistration.Dynamic dynamicFilter = servletContext
                .addFilter("loggingFilter", LoggingFilter.class);
        dynamicFilter.addMappingForUrlPatterns(
                EnumSet.of(DispatcherType.REQUEST), true, "/*");
    }
}

// 动态注册的Servlet类
@WebServlet
public class DynamicServlet extends HttpServlet {
    // Servlet实现
}
```

## 总结对比

| 注册方式          | 适用场景            | 优点             | 注意事项                        |
|---------------|-----------------|----------------|-----------------------------|
| Spring Bean注册 | 需要与Spring配置深度集成 | 配置集中管理，支持条件化注册 | 需要在配置类中显式声明                 |
| 注解声明扫描        | 简单的Servlet组件    | 声明简单，接近传统方式    | 需要`@ServletComponentScan`支持 |
| 动态编程注册        | 运行时动态调整         | 灵活性最高，可编程控制    | 需了解Servlet API细节            |

三种方案各有适用场景，你可以根据项目需求选择最合适的集成方式。掌握这些技能，无论是维护遗留系统还是构建新应用，都能更加得心应手。