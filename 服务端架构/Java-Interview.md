# Awesome Java Interview | Java 面试题集锦

# 语法基础

## 类与对象

# 数据结构

## 基础类型

- 写出下面代码的输出结果：

```java
public static void main(String[] args) {
    String str1 = "Hello,World";
    String str2 = "Hello,World";
    String str3 = new String("Hello,World");
    String str4 = str3.intern();

    System.out.println("str1 == str2 ? " + (str1 == str2));
    System.out.println("str2 == str3 ? " + (str2 == str3));
    System.out.println("str2 == str4 ? " + (str2 == str4));
}
```

# 并发编程

## 锁

## 同步

- CountDownLatch、CyclicBarrier、Semaphore 共同之处与区别以及各自使用场景？

## NIO

## Akka

# Spring

## IoC

- 为什么不推荐使用 Autowired？在多线程环境下为何有时候 Autowired 会无效？应该如何保证多线程下的注入？

## MyBatis

## Hibernate

- 简述下 Hibernate 的优劣特性。

# Toolkits & Framework

# JVM

## 类

- ClassLoader 的作用是什么？类加载的流程包含哪些步骤？
