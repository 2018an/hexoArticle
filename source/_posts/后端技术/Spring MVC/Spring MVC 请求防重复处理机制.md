---
title: Spring MVC 请求防重复处理机制
date: 2025-10-29 17:30:25
category: 后端
tags: Spring MVC
---

在Web应用中，防止用户重复提交表单是一个常见需求。Spring MVC框架可通过拦截器与令牌验证机制来有效解决这一问题。

定义自定义注解
首先创建一个自定义注解，用于标记需要处理重复请求的方法：

java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestToken {
// 是否生成令牌
boolean generate() default false;
// 是否验证并清除令牌
boolean verify() default false;
}
应用方式：

在页面渲染的方法上添加：@RequestToken(generate = true)

在处理提交请求的方法上添加：@RequestToken(verify = true)

实现请求拦截器
创建拦截器来统一处理令牌逻辑：

java
public class DuplicateRequestInterceptor extends HandlerInterceptorAdapter {

    private static final String REQUEST_TOKEN_KEY = "request_token";
    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
            Object handler) throws Exception {
        if (handler instanceof HandlerMethod) {
            Method method = ((HandlerMethod) handler).getMethod();
            RequestToken tokenAnnotation = method.getAnnotation(RequestToken.class);
            if (tokenAnnotation != null) {
                HttpSession session = request.getSession();
                
                // 生成令牌阶段
                if (tokenAnnotation.generate()) {
                    String tokenValue = UUID.randomUUID().toString();
                    session.setAttribute(REQUEST_TOKEN_KEY, tokenValue);
                    return true;
                }
                
                // 验证令牌阶段
                if (tokenAnnotation.verify()) {
                    if (isDuplicateRequest(request)) {
                        logger.warn("检测到重复请求，路径：" + request.getRequestURI());
                        response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
                        return false;
                    }
                    session.removeAttribute(REQUEST_TOKEN_KEY);
                }
            }
        }
        return super.preHandle(request, response, handler);
    }
    
    /**
     * 判断是否为重复请求
     */
    private boolean isDuplicateRequest(HttpServletRequest request) {
        HttpSession session = request.getSession();
        String serverToken = (String) session.getAttribute(REQUEST_TOKEN_KEY);
        
        // 会话中不存在令牌
        if (serverToken == null) {
            return true;
        }
        
        // 请求中未携带令牌
        String clientToken = request.getParameter(REQUEST_TOKEN_KEY);
        if (clientToken == null) {
            return true;
        }
        
        // 令牌不匹配
        return !serverToken.equals(clientToken);
    }

}
配置拦截器链
在Spring配置文件中注册拦截器：

xml
<!-- 配置拦截器链 -->
<mvc:interceptors>
<mvc:interceptor>
<mvc:mapping path="/**"/>
<bean class="com.example.web.interceptor.DuplicateRequestInterceptor"/>
</mvc:interceptor>
</mvc:interceptors>
前端表单集成
在视图模板的表单中添加令牌字段：

html
<input type="hidden" name="request_token" value="${session.getAttribute('request_token')}"/>
提交表单时，此令牌将随请求一同发送至服务器进行验证。