---
title: 掌握 Spring 框架这 10 个扩展点，开发效率直接翻倍
date: 2025-11-10 17:30:25
category: 后端
tags: Spring
---


当我们提到 Spring 时，或许首先映入脑海的是 IOC（控制反转）和 AOP（面向切面编程）。它们可以被视为 Spring 的基石。正是凭借其出色的设计，Spring 才能在众多优秀框架中脱颖而出。

Spring 具有很强的扩展性。许多第三方应用程序，如 rocketmq、mybatis、redis 等，都可以轻松集成到 Spring 系统中。让我们一起来看看 Spring 中最常用的十个扩展点。

1\. 全局异常处理
-------------------------------------------------------------------------------------------------------------------------------------------------

过去，在开发接口时，如果发生异常，我们通常需要给用户一个更友好的提示。但如果不进行错误处理，例如：

```
@RequestMapping("/test")
@RestController
publicclassTestController {
    @GetMapping("/division") publicStringdivision(@RequestParam("a") inta, @RequestParam("b") intb) {
        returnString.valueOf(a / b);
    }
}
```

这是一个计算 `a/b` 结果的方法，通过`127.0.0.1:8080/test/division?a=10&b=0` 访问后会出现以下结果：

![图片](img/2025111001.png)

什么？用户能直接看到如此详细的错误信息吗？

这种报错方式给用户带来了非常糟糕的体验。为了解决这个问题，我们通常在接口中捕获异常。

```
@GetMapping("/division")
publicString

division(@RequestParam("a")inta, @RequestParam("b")intb) {
    String result = "";
    try {
        result = String.

                valueOf(a / b);
    } catch (
            ArithmeticException e) {
        result = "params error";
    }
    returnresult;
}   

```

接口改造后，当发生异常时，会提示：“`params error`”，用户体验会更好。

如果只是一个接口，那没问题。但如果项目中有成百上千个接口，我们是否需要为所有接口添加异常处理代码呢？

肯定不能这样做的。这时，全局异常处理就派上用场了：`RestControllerAdvice`。

```
@RestControllerAdvice
publicclass GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    publicString handleException(Exceptione) {
        if (einstanceofArithmeticException) {
            return "params error";
        }
        if (einstanceofException) {
            return "Internal server exception";
        }
        returnnull;
    }
}   

```

只需在 `handleException` 方法中处理异常情况。业务接口可以放心使用，不再需要捕获异常（遵循统一的处理逻辑）。


2\. 自定义拦截器
-------------------------------------------------------------------------------------------------------------------------------------------------

与 Spring 拦截器相比，Spring MVC 拦截器可以在内部获取 `HttpServletRequest` 和 `HttpServletResponse` 等 Web 对象实例。

Spring MVC 拦截器的顶级接口是：`HandlerInterceptor`，它包含三个方法：

*   `preHandle`：在目标方法执行前执行。

*   `postHandle`：在目标方法执行后执行。

*   `afterCompletion`：在请求完成时执行。


为了方便起见，在一般情况下，我们通常使用 `HandlerInterceptor` 接口的实现类 `HandlerInterceptorAdapter`。

如果存在权限认证、日志记录和统计等场景，可以使用此拦截器。

第一步，通过继承 `HandlerInterceptorAdapter` 类定义一个拦截器：

```
public class AuthInterceptor extends HandlerInterceptorAdapter {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String requestUrl = request.getRequestURI();
        if (checkAuth(requestUrl)) {
            returntrue;
        }
        returnfalse;
    }

    private boolean checkAuth(String requestUrl) {
        System.out.println("===Authority Verification===");
        returntrue;
    }
}   
```

第二步，在 Spring 容器中注册此拦截器。

```
@Configuration
public class WebAuthConfig extends WebMvcConfigurerAdapter {
    @Bean
    public AuthInterceptor getAuthInterceptor() {
        return new AuthInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuthInterceptor());
    }
}   
```

随后，当请求接口时，Spring MVC 可以通过此拦截器自动拦截接口并验证权限。


3\. 获取 Spring 容器对象
---------------------------------------------------------------------------------------------------------------------------------------------------------

在日常开发中，我们经常需要从 Spring 容器中获取 Beans。但是你知道如何获取 Spring 容器对象吗？

#### 3.1 BeanFactoryAware 接口

```
@Service
public class StudentService implements BeanFactoryAware {
    private BeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    public void add() {
        Student student = (Student) beanFactory.getBean("student");
    }
}
```

实现 `BeanFactoryAware` 接口，然后重写 `setBeanFactory` 方法。从这个方法中，可以获取 Spring 容器对象。

#### 3.2 ApplicationContextAware 

```
@Service
public class StudentService2 implements ApplicationContextAware {
    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    public void add() {
        Student student = (Student) applicationContext.getBean("student");
    }
}   
```

4\. 导入配置
-----------------------------------------------------------------------------------------------------------------------------------------------

有时我们需要在某个配置类中导入其他一些类，并且导入的类也会被添加到 Spring 容器中。此时，可以使用`@Import` 注解来完成此功能。

如果你看过它的源代码，会发现导入的类支持三种不同的类型。

然而，我认为最好将普通类和带有`@Configuration` 注解的配置类分开解释。因此，列出了四种不同的类型：

#### 4.1 导入普通类

这种导入方式最简单。导入的类将被实例化为一个 bean 对象。

```
public class A {
}

@Import(A.class)
@Configuration
public class TestConfiguration {
}   
```

通过`@Import` 注解导入类 A，Spring 可以自动实例化对象 A。然后，可以在需要的地方通过`@Autowired` 注解进行注入：

```
@Autowired   
private A a;   
```

是不是很神奇？不需要添加`@Bean` 注解就可以实例化对象。

#### 4.2 导入带有@Configuration 注解的配置类

这种导入方式最复杂，因为`@Configuration` 注解还支持多种组合注解，例如：

*   @Import

*   @ImportResource

*   @PropertySource 等


```
public class A {
}

publicclass B {}

@Import(B.class)
@Configuration
public class AConfiguration {
    @Bean
    public A a() {
        returnnew A ();
    }
}

@Import(AConfiguration.class)
@Configuration
public class TestConfiguration {
}   
```

通过`@Import` 注解导入一个带有`@Configuration` 注解的配置类，与该配置类相关的`@Import`、`@ImportResource` 和`@PropertySource` 等注解导入的所有类将一次性全部导入。

#### 4.3 ImportSelector

这种导入方式需要实现 `ImportSelector` 接口：

```
public class AImportSelector implements ImportSelector {
    private static final String CLASS_NAME = "com.demo.cache.service.A";

    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{CLASS_NAME};
    }
}

@Import(AImportSelector.class)
@Configuration
public class TestConfiguration {
}   
```

这种方法的优点是 `selectImports` 方法返回一个数组，这意味着可以非常方便的导入多个类。

#### 4.4 ImportBeanDefinitionRegistrar

这种导入方式需要实现 `ImportBeanDefinitionRegistrar` 接口：

```
public class AImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(A.class);
        registry.registerBeanDefinition("a", rootBeanDefinition);
    }
}

@Import(AImportBeanDefinitionRegistrar.class)
@Configuration
public class TestConfiguration {
}   
```

5\. 项目启动时的附加功能
-----------------------------------------------------------------------------------------------------------------------------------------------------

有时我们需要在项目启动时自定义一些附加逻辑，例如加载一些系统参数、资源初始化、预热本地缓存等。我们该怎么做呢？Spring Boot 提供了两个接口来帮助我们实现上述要求：

*   CommandLineRunner

*   ApplicationRunner


它们的用法非常简单。以 `ApplicationRunner` 接口为例：

```
@Component
publicclass MyApplicationRunner implements

ApplicationRunner {
    @Override public void run (ApplicationArguments args) throws Exception
    {           // 在这里编写项目启动时需要执行的代码       
        System.out.println("项目启动时执行附加功能，加载系统参数...");
        // 假设这里从配置文件中加载系统参数并进行处理    
        Properties properties = new Properties();
        try (InputStream inputStream = new FileInputStream("application.properties")) {
            properties.load(inputStream);
            String systemParam = properties.getProperty("system.param");
            System.out.println("加载的系统参数值为：" + systemParam);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}   
```

在上述代码中，我们实现了 `ApplicationRunner` 接口，并重写了 run 方法。在 run 方法中，我们可以编写在项目启动时需要执行的附加功能代码，例如加载系统参数、初始化资源、预热缓存等。

这里只是简单地模拟了从配置文件中加载系统参数并打印出来，实际应用中可以根据具体需求进行更复杂的操作。

当项目启动时，Spring Boot 会自动检测并执行实现了 `ApplicationRunner` 或 `CommandLineRunner` 接口的类中的 run 方法，从而实现项目启动时的附加功能。

这两个接口的区别在于参数类型不同，`ApplicationRunner` 的 run 方法参数是 `ApplicationArguments`，它提供了更多关于应用程序参数的信息，而 `CommandLineRunner` 的 run 方法参数是原始的字符串数组，直接包含了命令行参数。根据具体需求可以选择使用其中一个接口来实现项目启动时的附加功能。

6\. 修改 BeanDefinition
------------------------------------------------------------------------------------------------------------------------------------------------------------

在实例化 Bean 对象之前，Spring IOC 需要先读取 Bean 的相关属性，将它们保存在 `BeanDefinition` 对象中，然后通过 `BeanDefinition` 对象实例化 Bean 对象。

如果你想修改 `BeanDefinition` 对象中的属性，该怎么做呢？我们可以实现 `BeanFactoryPostProcessor` 接口。

```
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        DefaultListableBeanFactory defaultListableBeanFactory = (DefaultListableBeanFactory) configurableListableBeanFactory;
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(User.class);
        beanDefinitionBuilder.addPropertyValue("id", 123);
        beanDefinitionBuilder.addPropertyValue("name", "Dylan Smith");
        defaultListableBeanFactory.registerBeanDefinition("user", beanDefinitionBuilder.getBeanDefinition());
    }
}   
```

在 `postProcessBeanFactory` 方法中，可以获取 `BeanDefinition` 的相关对象并修改该对象的属性。

7\. 初始化方法
------------------------------------------------------------------------------------------------------------------------------------------------

目前，Spring 中比较常用的初始化 bean 的方法有：

*   使用`@PostConstruct` 注解。

*   实现 `InitializingBean` 接口。


#### 7.1 使用@PostConstruct 注解

```
@Service
public class AService {
    @PostConstruct
    public void init() {
        System.out.println("===Initializing===");
    }
}   
```

在需要初始化的方法上添加`@PostConstruct` 注解。这样，它就具有了初始化的能力。

#### 7.2 实现 InitializingBean 接口

```
@Service
public class BService implements InitializingBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("===Initializing===");
    }
}   
```

8\. 在初始化 Bean 前后添加逻辑
-----------------------------------------------------------------------------------------------------------------------------------------------------------

有时，你希望在初始化 bean 之前和之后实现一些自己的逻辑。

这时，可以实现 `BeanPostProcessor` 接口。

这个接口目前有两个方法：

*   `postProcessBeforeInitialization`：在初始化方法之前调用。

*   `postProcessAfterInitialization`：在初始化方法之后调用。


例如：

```
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof User) {
            ((User) bean).setUserName("Dylan Smith");
        }
        return bean;
    }
}   
```

如果 Spring 中有一个 User 对象，将其 `userName` 设置为：`Dylan Smith`。

实际上，我们经常使用的注解，如`@Autowired`、`@Value`、`@Resource`、`@PostConstruct` 等，都是通过 `AutowiredAnnotationBeanPostProcessor` 和 `CommonAnnotationBeanPostProcessor` 实现的。

9\. 在关闭容器之前添加操作
------------------------------------------------------------------------------------------------------------------------------------------------------

有时，我们需要在关闭 Spring 容器之前做一些额外的工作，例如关闭资源文件。

这时，我们可以实现 `DisposableBean` 接口并覆盖其 `destroy` 方法：

```
@Service
public class DService implements InitializingBean, DisposableBean {
    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean destroy");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean afterPropertiesSet");
    }
}   
```

这样，在 Spring 容器销毁之前会调用 destroy 方法。通常，我们会同时实现 `InitializingBean` 和 `DisposableBean` 接口，并覆盖初始化方法和销毁方法。

10\. 自定义作用域
--------------------------------------------------------------------------------------------------------------------------------------------------

**我们都知道，Spring 只支持两种默认的 Scope：**

*   `singleton`：在单例作用域中，从 Spring 容器中获取的每个 bean 都是同一个对象。

*   `prototype`：在原型作用域中，从 Spring 容器中获取的每个 bean 都是不同的对象。


**Spring Web 扩展了 Scope 并添加了：**

*   `RequestScope`：在同一个请求中，从 Spring 容器中获取的 bean 都是同一个对象。

*   `SessionScope`：在同一个会话中，从 Spring 容器中获取的 bean 都是同一个对象。


即便如此，有些场景仍然无法满足我们的要求。

例如，如果我们希望在同一个线程中从 Spring 容器中获取的所有 bean 都是同一个对象，该怎么办呢？

这就需要自定义 Scope。

##### 第一步，实现 Scope 接口：

```
public class ThreadLocalScope implements Scope {
    privatestaticfinal ThreadLocal
    THREAD_LOCAL_SCOPE =new

    ThreadLocal();

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Object value = THREAD_LOCAL_SCOPE.get();
        if (value != null) {
            return value;
        }
        Object object = objectFactory.getObject();
        THREAD_LOCAL_SCOPE.set(object);
        return object;
    }

    @Override
    public Object remove(String name) {
        THREAD_LOCAL_SCOPE.remove();
        returnnull;
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
    }

    @Override
    public Object resolveContextualObject(String key) {
        returnnull;
    }

    @Override
    public String getConversationId() {
        returnnull;
    }
}   
```

##### 第二步，将新定义的“Scope”注入到 Spring 容器中：

```
@Component
public class ThreadLocalBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        beanFactory.registerScope("threadLocalScope", new ThreadLocalScope());
    }
}   
```

第三步，使用新定义的“Scope”：

```
@Scope("threadLocalScope")
@Service
public class CService {
    public void add() {
    }
}   
```

总结
-----------------------------------------------------------------------------------------------------------------------------------------

好了，今天的内容就到这里。对 Spring 框架感兴趣的读者可以关注我，后续会分享更多有关 Spring 的相关知识。