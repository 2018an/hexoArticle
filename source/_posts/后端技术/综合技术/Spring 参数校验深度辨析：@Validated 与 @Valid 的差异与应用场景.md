---
title: Spring 参数校验深度辨析：@Validated 与 @Valid 的差异与应用场景
date: 2025-10-29 17:30:25
category: 后端
tags: 综合技术
---

在 Spring MVC 开发中，使用 Bean Validation 进行请求参数校验是保证接口健壮性的重要手段。@Valid (JSR-303/JSR-380 标准) 和
@Validated (Spring 框架的扩展) 是两个最常用的注解，它们功能相似但存在关键区别，尤其是在分组校验和嵌套校验的场景下。

核心区别一览
特性 @Valid (javax.validation.Valid)    @Validated (org.springframework.validation.annotation.Validated)
来源 Java EE / Jakarta EE 标准 (JSR)    Spring 框架提供的扩展
分组校验 不支持 支持。这是其最大优势，可通过 groups 属性指定校验组。
可注解位置 方法、参数、字段、容器元素。 类、方法、参数。不能直接注解在字段上。
嵌套校验触发 支持。在类的字段上标注 @Valid，可触发该字段内部属性的校验。 本身不直接支持。需依赖字段上的 @Valid 来配合触发嵌套校验。
Spring 特性集成 基础支持 与 Spring 的方法级验证、AOP 等特性集成更好。
关键差异深度解析

1. 分组校验（Group Validation）
   这是选择 @Validated 的首要理由。它允许你根据不同的业务场景（如新增、更新）应用不同的校验规则。

java
// 定义校验组
public interface CreateGroup {}
public interface UpdateGroup {}

public class UserDTO {
@NotNull(groups = {CreateGroup.class, UpdateGroup.class})
private Long id;

    @NotBlank(groups = CreateGroup.class) // 仅创建时需要
    private String username;

    @Email(groups = {CreateGroup.class, UpdateGroup.class})
    private String email;

}

@RestController
public class UserController {
// 创建用户时，校验CreateGroup组
@PostMapping("/users")
public void createUser(@Validated(CreateGroup.class) @RequestBody UserDTO user) {
// ...
}

    // 更新用户时，校验UpdateGroup组
    @PutMapping("/users/{id}")
    public void updateUser(@Validated(UpdateGroup.class) @RequestBody UserDTO user) {
        // ...
    }

}
@Valid 注解没有 groups 属性，无法实现此类精细控制。

2. 嵌套校验（Nested Validation）
   当对象属性包含另一个需要校验的对象时，需要触发嵌套校验。这里的组合使用是关键。

java
public class OrderDTO {
@NotNull
private String orderNo;

    // 关键：在字段上使用 @Valid 来声明“此字段内部也需要校验”
    @Valid // 必须使用 @Valid
    @NotNull
    private UserDTO user; // UserDTO内部也有自己的校验注解

    @Valid // 同样，集合内的元素也需要校验
    @NotEmpty
    private List<@Valid ItemDTO> items;

}

@RestController
public class OrderController {
// 在Controller参数上，使用 @Validated 或 @Valid 均可触发第一层校验，
// 并借助字段上的 @Valid 进行嵌套校验。
@PostMapping("/orders")
public void createOrder(@Valid @RequestBody OrderDTO order) { // 这里用 @Valid 也可以
// 会校验 OrderDTO 本身，以及其内部的 user 和 items 字段
}
}
核心要点：

@Validated 或 @Valid 用在方法参数上，仅触发当前对象的直接属性校验。

要触发嵌套校验，必须在类的成员字段（或集合泛型参数）上明确添加 @Valid 注解。由于 @Validated 不能注解在字段上，因此嵌套校验必须依赖
@Valid。

在 Controller 参数位置，两者可以互换，但通常使用 @Valid 以保持标准，或使用 @Validated 以启用分组功能。

最佳实践建议
需要分组校验时：在 Controller 方法参数上，必须使用 @Validated(Group.class)。

需要嵌套校验时：在类的成员字段（对象或集合）上，必须使用 @Valid 来触发。

普通单层校验时：两者在 Controller 参数上作用几乎相同。为了一致性和标准性，可优先使用 @Valid。如果项目已广泛使用 Spring 特性，使用
@Validated 也无妨。

方法级校验（Service层）：在 Spring 管理的 Service 方法上进行参数校验时，必须使用 @Validated 注解在类上，然后在方法参数上使用
@Valid 或其他约束注解。