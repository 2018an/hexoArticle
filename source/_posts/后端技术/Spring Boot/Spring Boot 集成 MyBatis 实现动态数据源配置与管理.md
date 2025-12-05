---
title: Spring Boot 集成 MyBatis 实现动态数据源配置与管理
date: 2025-10-29 17:30:25
category: 后端
tags: Spring Boot
---

## 构建高可用数据访问层：Spring Boot动态多数据源实践

在现代分布式应用架构中，数据库读写分离是提升系统性能与可用性的重要策略。本文将深入探讨如何基于Spring
Boot框架，整合MyBatis与动态数据源技术，构建灵活高效的多数据源访问方案。

### 项目依赖配置

首先在项目构建文件中引入必要的技术组件：

```xml
<!-- MyBatis Spring Boot启动器 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
</dependency>

        <!-- 阿里巴巴Druid高性能连接池 -->
<dependency>
<groupId>com.alibaba</groupId>
<artifactId>druid</artifactId>
</dependency>

        <!-- Oracle数据库驱动（根据实际数据库调整） -->
<dependency>
<groupId>com.oracle</groupId>
<artifactId>ojdbc6</artifactId>
</dependency>
```

### 应用启动入口配置

创建应用主启动类，并进行关键配置：

```java

@EnableMyBatisIntegration
@EnableTransactionManagement
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class DataServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(DataServiceApplication.class, args);
    }
}
```

**关键注解说明**：

- `@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })`：排除Spring Boot默认的单数据源自动配置，为自定义多数据源方案让路
- `@EnableTransactionManagement`：启用声明式事务管理支持
- `@EnableMyBatisIntegration`：自定义注解，用于激活MyBatis集成配置

自定义的`@EnableMyBatisIntegration`注解定义如下：

```java

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(MyBatisConfiguration.class)
public @interface EnableMyBatisIntegration {
}
```

### MyBatis核心配置实现

创建MyBatis的主配置类，负责数据源路由和会话工厂的构建：

```java

@Configuration
@MapperScan(basePackages = DataSourceConstants.MAPPER_PACKAGES)
public class MyBatisConfiguration implements DataSourceConstants {

    @Primary
    @Bean
    public DynamicDataSourceRouter dynamicDataSource(
            @Qualifier(MASTER_DB) DataSource masterDataSource,
            @Qualifier(SLAVE_DB) DataSource slaveDataSource) {

        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put(MASTER_DB, masterDataSource);
        dataSourceMap.put(SLAVE_DB, slaveDataSource);

        DynamicDataSourceRouter router = new DynamicDataSourceRouter();
        router.setDefaultTargetDataSource(masterDataSource);
        router.setTargetDataSources(dataSourceMap);
        return router;
    }

    @Bean
    public PlatformTransactionManager transactionManager(DynamicDataSourceRouter dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory(DynamicDataSourceRouter dataSource) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        factoryBean.setMapperLocations(
                new PathMatchingResourcePatternResolver()
                        .getResources(DataSourceConstants.MAPPER_LOCATIONS));
        return factoryBean.getObject();
    }
}
```

相关常量定义接口：

```java
public interface DataSourceConstants {

    String CONFIG_PREFIX = "spring.datasource";
    String ACTIVE_PROFILE = "active";

    String MASTER_DB = "db-master";
    String SLAVE_DB = "db-slave";

    String DRUID_POOL = "druid";

    String MAPPER_PACKAGES = "com.example.**.dao";
    String MAPPER_LOCATIONS = "classpath:mapper/**/*.xml";
}
```

### 数据库连接池精细化配置

针对Druid连接池的自动配置类，实现主从数据源的独立参数管理：

```java

@Configuration
@ConditionalOnClass(DruidDataSource.class)
@ConditionalOnProperty(prefix = DataSourceConstants.CONFIG_PREFIX,
        value = DataSourceConstants.ACTIVE_PROFILE,
        havingValue = DataSourceConstants.DRUID_POOL)
public class DruidAutoConfiguration implements DataSourceConstants {

    private final Logger logger = LoggerFactory.getLogger(getClass());

    @Bean(name = MASTER_DB, initMethod = "init", destroyMethod = "close")
    public DataSource masterDataSource(MasterDataSourceProperties properties) throws SQLException {
        logger.info("初始化主数据源，连接地址: {}", properties.getUrl());

        DruidDataSource ds = new DruidDataSource();
        // 设置连接池核心参数
        ds.setUrl(properties.getUrl());
        ds.setUsername(properties.getUsername());
        ds.setPassword(properties.getPassword());
        ds.setDriverClassName(properties.getDriverClassName());

        // 连接池性能调优参数
        ds.setInitialSize(properties.getInitialSize());
        ds.setMinIdle(properties.getMinIdle());
        ds.setMaxActive(properties.getMaxActive());
        ds.setMaxWait(properties.getMaxWait());

        // 连接有效性检测配置
        ds.setValidationQuery(properties.getValidationQuery());
        ds.setTestWhileIdle(properties.isTestWhileIdle());
        ds.setTestOnBorrow(properties.isTestOnBorrow());

        return ds;
    }

    @Bean(name = SLAVE_DB, initMethod = "init", destroyMethod = "close")
    public DataSource slaveDataSource(SlaveDataSourceProperties properties) throws SQLException {
        logger.info("初始化从数据源，连接地址: {}", properties.getUrl());

        DruidDataSource ds = new DruidDataSource();
        // 配置从库连接池参数，可与主库参数区分
        ds.setUrl(properties.getUrl());
        ds.setUsername(properties.getUsername());
        ds.setPassword(properties.getPassword());
        // ... 其他参数配置
        return ds;
    }

    // Druid监控面板Servlet注册
    @Bean
    public ServletRegistrationBean<StatViewServlet> druidStatViewServlet() {
        ServletRegistrationBean<StatViewServlet> registration =
                new ServletRegistrationBean<>(new StatViewServlet(), "/druid/*");
        registration.addInitParameter("loginUsername", "admin");
        registration.addInitParameter("loginPassword", "admin123");
        return registration;
    }
}
```

### 智能数据源路由机制

**1. 数据源选择注解**
定义用于标记数据源选择的注解：

```java

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DataSourceSelector {
    String value() default DataSourceConstants.MASTER_DB;
}
```

**2. 动态数据源路由器**
继承Spring的`AbstractRoutingDataSource`实现智能路由：

```java
public class DynamicDataSourceRouter extends AbstractRoutingDataSource {

    private final Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    protected Object determineCurrentLookupKey() {
        String selectedDataSource = DataSourceContextManager.getCurrentDataSource();
        logger.debug("当前线程使用数据源: {}", selectedDataSource);
        return selectedDataSource;
    }
}
```

**3. 基于AOP的数据源切换**
通过切面编程实现方法级别的数据源动态切换：

```java

@Aspect
@Component
public class DataSourceRoutingAspect {

    @Before("@annotation(DataSourceSelector)")
    public void switchDataSource(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        DataSourceSelector selector = method.getAnnotation(DataSourceSelector.class);

        if (selector != null) {
            DataSourceContextManager.setDataSource(selector.value());
        }
    }

    @After("@annotation(DataSourceSelector)")
    public void restoreDataSource() {
        DataSourceContextManager.clearDataSource();
    }
}
```

**4. 线程上下文管理器**
使用ThreadLocal保证多线程环境下的数据源隔离：

```java
public class DataSourceContextManager {

    private static final ThreadLocal<String> CONTEXT_HOLDER = new ThreadLocal<>();

    public static void setDataSource(String dbName) {
        CONTEXT_HOLDER.set(dbName);
    }

    public static String getCurrentDataSource() {
        return CONTEXT_HOLDER.get() != null ?
                CONTEXT_HOLDER.get() : DataSourceConstants.MASTER_DB;
    }

    public static void clearDataSource() {
        CONTEXT_HOLDER.remove();
    }
}
```

### 实际应用示例

在业务层使用数据源选择注解：

```java

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper userMapper;

    @Override
    @DataSourceSelector(DataSourceConstants.MASTER_DB)
    @Transactional
    public void createUser(User user) {
        // 写操作使用主库
        userMapper.insert(user);
    }

    @Override
    @DataSourceSelector(DataSourceConstants.SLAVE_DB)
    public User getUserById(Long id) {
        // 读操作使用从库
        return userMapper.selectById(id);
    }
}
```

### 总结

通过以上架构设计，我们实现了：

1. **透明化数据源切换**：业务代码无需关心具体数据源连接细节
2. **读写操作自动分离**：基于注解声明自动路由到合适的数据源
3. **线程安全的数据源管理**：通过ThreadLocal保证多线程环境下的数据源隔离
4. **灵活的扩展能力**：可轻松扩展支持更多数据源或复杂路由策略

这种方案特别适用于需要高并发读访问的场景，通过将读压力分散到多个从库，显著提升系统整体吞吐量和可用性。同时，由于切换逻辑对业务透明，极大降低了代码复杂性和维护成本。