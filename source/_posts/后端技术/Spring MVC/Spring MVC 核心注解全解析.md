---
title: Spring MVC 核心注解全解析
date: 2025-10-29 17:30:25
category: 后端
tags: Spring MVC
---

Spring MVC框架提供了一系列强大的注解，极大简化了Web应用的开发。下面将详细介绍这些核心注解的功能与用法。

核心注解详解
@Controller

标记一个类为控制器组件，Spring MVC会自动扫描并管理带有此注解的类。

@RequestMapping

用于映射HTTP请求到特定的处理方法。可应用于类级别或方法级别，可指定HTTP方法、请求参数等条件。

@RequestParam

标注方法参数，表示从请求参数中获取值。主要用于处理application/x-www-form-urlencoded格式的数据。

@RequestBody

标注方法参数，指示Spring MVC从请求体中读取数据并绑定到参数对象。适用于接收JSON、XML等格式的非表单数据。

@ResponseBody

标注方法或返回类型，表明方法返回值应直接写入HTTP响应体，而非进行视图解析。常用于RESTful API返回JSON/XML数据。

@RestController

结合了@Controller和@ResponseBody的功能，标记的类中所有方法返回值都将直接写入响应体。

@PathVariable

从URL路径模板中提取变量值并绑定到方法参数，支持RESTful风格URL。

@RequestHeader

将HTTP请求头中的值绑定到方法参数。

@CookieValue

将HTTP请求中的Cookie值绑定到方法参数。

HTTP方法特定注解

Spring 4.3+引入了@GetMapping、@PostMapping、@PutMapping、@DeleteMapping等注解，它们是对@RequestMapping的语义化封装，使代码更具可读性。

实践示例
java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/api")
public class ApiController {

    /**
     * RESTful风格GET请求示例
     */
    @GetMapping("/users/{id}")
    @ResponseBody
    public User getUserById(@PathVariable("id") Long userId) {
        return userService.findUserById(userId);
    }
    
    /**
     * POST请求接收JSON数据示例
     */
    @PostMapping("/users")
    @ResponseBody
    public ApiResponse createUser(@RequestBody User user) {
        userService.saveUser(user);
        return ApiResponse.success("创建成功");
    }
    
    /**
     * 获取请求头信息示例
     */
    @GetMapping("/info")
    @ResponseBody
    public String getRequestInfo(@RequestHeader("User-Agent") String userAgent) {
        return "客户端信息：" + userAgent;
    }

}