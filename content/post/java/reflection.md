---
title: "Java必备知识：反射"
date: 2018-02-16T10:25:12+08:00
categories: ["技术之美"]
tags: ["Java"]
draft: false
toc: true
summary: "反射在Java框架中的使用非常频繁，是我们进阶路上非常重要的一块垫脚石。"
---

带着问题思考：

- 如何理解反射？反射的机制？
- 如何使用反射？
- 反射如何提高一个`Java`框架的效率？



## 如何理解反射？反射的机制？

什么是反射？

反射就是**将类的各个组成部分封装为其他对象**, 方便我们在**程序运行时使用这些对象**。反射有利于程序的解耦，提高程序的可拓展性。不和你多BB，上图：

![](../index.assets/reflection_states.png)



## 如何使用反射？

#### 获取`Class`对象的三种方式

在上图所示的不同阶段，我们有不同的获取方式：

- 源代码阶段：`Class.forName("全类名")  //将字节码加载进内存，返回Class对象`
- `Class`类对象阶段： `类名.class`，这是通过类名的属性 `class`获取的。
- 运行时阶段：`对象.getClass()`，这个方法是在`Object`类中定义的。

```java
Class cls1 = Class.forName("完整类名")；

Class cls2 = String.class;

String string = "";
Class cls3 = string.getClass();

//同一个字节码文件（*.class）在一次运行过程中，用以上三种方法获取的对象都是相同的。
System.out.println(cls1 == cls2); //true
System.out.println(cls1 == cls3); //true
```



#### Class文件中包含的信息

1. 类名
2. 成员变量
3. 构造方法
4. 成员方法

具体的`API`就不列出来了，其中注意`API`中带`Declared`和不带的区别： 

- 不带：获取带 `public`修饰符
- 带：   获取所有修饰符

访问`privated`修饰符的变量或者方法时，需要忽略权限修饰符的安全检查，具体的代码操作：

```Java
Field d = personClass.getDeclaredField("age");
d.setAccessible(true); //忽略安全检查，参数和方法同理。

Object person = personClass.newInstance(); //class实例的空参构造方法
```

