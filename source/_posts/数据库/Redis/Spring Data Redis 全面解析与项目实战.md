---
title: Spring Data Redis 全面解析与项目实战
date: 2025-10-30 14:42:34
category: 数据库
tags: Redis
---

Spring Data Redis（简称 SDR）为 Spring 应用程序提供了便捷的 Redis 配置与访问能力。它提供了不同层次的抽象，使开发者能够专注于业务逻辑，而无需过多关注底层基础设施的细节。

Spring Boot 集成实战
添加项目依赖
```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-redis</artifactId>
<version>${spring-boot.version}</version>
</dependency>
```
配置 Redis 缓存与模板
```java
@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

    @Bean
    public KeyGenerator keyGenerator() {
        return (target, method, params) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getName());
            sb.append(method.getName());
            for (Object obj : params) {
                sb.append(obj.toString());
            }
            return sb.toString();
        };
    }

    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager manager = new RedisCacheManager(redisTemplate);
        manager.setDefaultExpiration(10000); // 默认过期时间10秒
        return manager;
    }

    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate template = new StringRedisTemplate(factory);
        // 设置值序列化器为JSON
        template.setValueSerializer(getJsonSerializer());
        template.afterPropertiesSet();
        return template;
    }

    private RedisSerializer<?> getJsonSerializer() {
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

        Jackson2JsonRedisSerializer<?> serializer = new Jackson2JsonRedisSerializer<>(Object.class);
        serializer.setObjectMapper(om);
        return serializer;
    }

}
```
配置连接参数
在 application.yml 中配置：

```yaml
spring.redis:
database: 0
host: 192.168.1.168
port: 6379
timeout: 0
pool:
max-active: 8
max-idle: 8
max-wait: -1
min-idle: 0
```
开始使用
在 Service 中注入 RedisTemplate 即可操作 Redis：

```java
@Autowired
private RedisTemplate redisTemplate;

public void someMethod() {
redisTemplate.opsForValue().set("currentTime", System.currentTimeMillis());
}
```
通过 RedisTemplate 操作数据
RedisTemplate 是 SDR 的核心类，它提供了高级抽象，负责序列化、连接管理等。此外，它还提供了一系列针对特定数据类型的“操作视图”（Operation
Views）。

数据类型操作视图：
接口 描述
GeoOperations 地理空间操作（GEOADD, GEORADIUS...）
HashOperations 哈希类型操作
HyperLogLogOperations HyperLogLog 操作（PFADD, PFCOUNT）
ListOperations 列表操作
SetOperations 集合操作
ValueOperations 字符串操作
ZSetOperations 有序集合操作
键绑定操作视图：
接口 描述
BoundGeoOperations 绑定键的地理空间操作
BoundHashOperations 绑定键的哈希操作
BoundKeyOperations 绑定键的通用操作
BoundListOperations 绑定键的列表操作
BoundSetOperations 绑定键的集合操作
BoundValueOperations 绑定键的字符串操作
BoundZSetOperations 绑定键的有序集合操作
使用示例：

```java
@Component
public class SomeService {
@Autowired
private RedisTemplate<String, String> template;

    @Resource(name="redisTemplate")
    private ListOperations<String, String> listOps;

    public void addItem(String userId, String item) {
        listOps.leftPush(userId, item);
    }

}
```
RedisTemplate 是线程安全的，可在多个地方复用。

RedisTemplate vs StringRedisTemplate
StringRedisTemplate 继承自 RedisTemplate。

StringRedisTemplate 默认使用字符串序列化器；RedisTemplate 默认使用 JDK 序列化。

两者序列化方式不同，数据不互通。推荐使用 StringRedisTemplate，除非需要存储复杂对象。

执行底层操作
可通过 execute 方法执行回调，进行底层操作，例如切换数据库：

```java
stringRedisTemplate.execute((RedisCallback<Boolean>) connection -> {
StringRedisConnection stringConn = (StringRedisConnection) connection;
stringConn.select(5); // 切换到5号数据库
stringConn.set("key", "value");
return true;
});
```
序列化器（Serializer）
SDR 使用序列化器在 Java 对象与 Redis 存储的字节之间进行转换。

常见实现
实现类 描述
StringRedisSerializer 字符串与字节数组转换，速度最快
JdkSerializationRedisSerializer JDK 默认序列化
OxmSerializer XML 序列化，体积大，速度慢
Jackson2JsonRedisSerializer JSON 序列化，需指定类型
GenericJackson2JsonRedisSerializer JSON 序列化，无需指定类型
对于简单字符串，使用 StringRedisSerializer；对于对象，推荐使用 JSON 序列化器。

事务支持
SDR 支持 Redis 的事务命令（multi, exec, discard），通过 SessionCallback 接口在同一连接中执行多个操作。

示例：

```java
List<Object> results = stringRedisTemplate.execute(new SessionCallback<List<Object>>() {
@Override
public List<Object> execute(RedisOperations operations) throws DataAccessException {
operations.multi();
operations.opsForSet().add("key1", "value1");
operations.opsForSet().add("key2", "value2");
return operations.exec(); // 提交事务
}
});
```
若事务中某条命令失败（如类型错误），则整个事务回滚。

声明式事务支持（@Transactional）
默认未启用，需在 RedisTemplate 上显式开启：template.setEnableTransactionSupport(true)。开启后，同一连接上的写操作会进入队列，读操作会转到新连接。需要配合
Spring 的 PlatformTransactionManager 使用。

配置示例：

```java
@Configuration
public class RedisTxConfig {
@Bean
public StringRedisTemplate redisTemplate() {
StringRedisTemplate template = new StringRedisTemplate(redisConnectionFactory());
template.setEnableTransactionSupport(true); // 启用事务支持
return template;
}
// ... 配置 TransactionManager 和 ConnectionFactory
}
```
注意事项：在 @Transactional 方法中，通过 RedisTemplate 执行的写操作在事务提交前，对同一连接的其他读操作不可见。