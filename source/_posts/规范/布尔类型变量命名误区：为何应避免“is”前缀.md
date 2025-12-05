---
title: 布尔类型变量命名误区：为何应避免“is”前缀
date: 2025-10-30 14:42:34
category: 程序人生
tags: 规范
---

最近又有同事来问我：为什么变量命名不推荐使用 isXXX 这种格式？这到底会引发什么问题？

这已经是多次被问及了。我通常建议对方查阅阿里巴巴的《Java开发手册》或自行搜索。实际上，具有一定经验的开发者都清楚这是一个基础规范。考虑到不少初学者可能也存在此疑问，我在此撰文进行说明。

首先，我们来看阿里巴巴《Java开发手册》中的相关规定：

【强制】POJO 类中布尔类型变量都不要加 is 前缀，否则部分框架解析会引起序列化错误。
反例：定义为基本数据类型 Boolean isDeleted 的属性，它的方法也是 isDeleted()，RPC 框架在反向解析的时候，“误以为”对应的属性名称是
deleted，导致属性获取不到，进而抛出异常。

以上规范明确指出，使用 isXXX 命名可能导致序列化过程中的解析异常。

通过一段 IDE 自动生成的 getter/setter 代码来进一步说明：

text
public class Staff {

    private String name;
    private boolean graduated;
    private boolean isMarried;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public boolean isGraduated() {
        return graduated;
    }

    public void setGraduated(boolean graduated) {
        this.graduated = graduated;
    }

    public boolean isMarried() {
        return isMarried;
    }

    public void setMarried(boolean married) {
        isMarried = married;
    }

}
对于变量 isMarried，IDE 生成的 getter 方法为 isMarried()，setter 为 setMarried()。部分序列化框架在解析时，可能会尝试寻找名为
married 的字段，从而导致属性无法正确映射。

而对于变量 graduated，虽然同为布尔类型，其生成的 getter 也是 isGraduated()，但由于其字段名本身不包含
“is”，因此不会引发上述框架解析歧义。这也正是规范建议避免以 “is” 作为布尔变量前缀的原因。

我曾有同事在使用某 Web 框架时遇到过此问题：在页面标签中尝试显示对象属性值时始终报错，最终定位正是由 isXXX 命名导致的字段映射失败。

今后若再遇到类似疑问，可将本文作为参考。