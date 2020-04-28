---
title: "Java必备知识：泛型"
date: 2018-02-15T11:26:28+08:00
categories: ["技术之美"]
tags: ["Java"]
draft: false
toc: true
summary: "泛型都不知道？源码怎么阅读？设计模式如何了解？当然，最主要的还是不开心。"
---

带着问题思考：

- 为什么使用泛型而不使用`Object`？

- 泛型中的`E`和`T`有什么区别？


- 如何使用泛型作为函数的返回类型？


- 如何定义泛型上限和下限？




## 为什么使用泛型而不使用`Object`？

我们先尝试下使用`Object` ，`Show me the fucking code`

```java
private static void show01() {
    ArrayList list = new ArrayList();
    list.add("abd");
    list.add(1);

    Iterator it = list.iterator();
    while (it.hasNext()) {
        Object obj = it.next();
        System.out.println("content:" + obj);

        //1.我们的需求：输出每个元素的字符长度：
        String str = (String) obj;
        System.out.println("length:" + str.length());
    }
}
```

问题：（代码不安全，而且产生运行期异常）当我们初始化一个集合时，**如果我们不指定其类型，系统会将元素类型默认为`Object`类型元素**，此时我们可以向集合添加任何元素，此时会造成安全问题。其中1处处理时不可将`Integer`强转为`String`类型。运行代码时会报错，但是不会产生任何语法错误。

使用泛型相信都很熟练了，就不过多演示代码。

泛型好处：1.避免类型转换。 2.  代码在编译期就可以发现问题。

泛型缺陷：只能向其添加定义类型。



## 泛型中的`E`和`T`有什么区别？

实际上只是一种使用习惯。

**E** - Element (在集合中使用，因为集合中存放的是元素)

**T** - Type（Java 类）

**K** - Key（键）

**V** - Value（值）

**N** - Number（数值类型）

**？** - 表示不确定的java类型



## 如何使用泛型作为函数的返回类型？

其实这是个很基础的问题。基础不牢，地动山摇！

`Code`：

```java
public <M> M GenericMethod(M m){
    System.out.println(m);
    return m;
}
```

其实`< M >`就是用来声明一个泛型类型的。我们在函数中任何地方（参数，修饰符）使用泛型时，我们都要使用`< M >`来声明泛型类型。



## 如何定义泛型上限和下限？

使用`extends`定义上限，`super`定义下限。

```java
public void getInstance(Class< ? extends Number> clz){
}
```

我们常常将泛型和反射结合起来，发挥更大的威力，例如`retrofit`网络框架。