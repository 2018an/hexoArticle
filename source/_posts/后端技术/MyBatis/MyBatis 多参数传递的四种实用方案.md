---
title: MyBatis 多参数传递的四种实用方案
date: 2025-10-29 17:30:25
category: 后端
tags: MyBatis
---

![image](img/18-1-20-16848962.jpg)

当前多数项目选择Mybatis作为持久层框架，部分企业仍沿用Hibernate。Mybatis的核心特点在于需要开发者手动编写SQL语句，而复杂业务场景下如何高效传递多个参数成为一项重要技能。

**以下是四种常用的多参数传递方案：**

**方案一：位置参数法**

```java
public User selectUser(String name, int deptId);
```

```xml

<select id="selectUser"
        resultMap="UserResultMap">
    select *
    from user
    where user_name = #{0}
    and dept_id = #{1}
</select>
```

#{}中的数字表示参数在方法中的位置序号。

此方法可读性较差，参数顺序变更易导致错误，不推荐使用。

方案二：注解标识法

```java
public User selectUser(@Param("userName") String name, @Param("deptId") int deptId);
```

```xml

<select id="selectUser" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

#{}中的名称需与@Param注解定义的参数标识保持一致。

参数数量较少时，此方法直观清晰，建议采用。

方案三：键值对映射法

```java
public User selectUser(Map<String, Object> params);
```

```xml

<select id="selectUser" parameterType="java.util.Map" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

#{}中的名称对应Map中的键名。

此方案适用于参数数量较多或需要动态传递的场景，灵活性较高。

方案四：对象封装法

```java
public User selectUser(User user);
```

```xml

<select id="selectUser" parameterType="com.test.User" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

#{}中的名称对应实体类的属性字段。

此方法语义明确，但需要预先定义实体类，扩展时需修改类结构，根据实际情况选用。

如果觉得本文有帮助，欢迎分享给更多开发者！