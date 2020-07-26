# Spring AOP

[TOC]

## AOP 是什么

AOP（Aspect-Oriented Programming，面向切面编程）能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

## Spring 如何实现 AOP

Spring AOP 的实现是基于动态代理的，有两种方式：

- 如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 JDK 中的 Proxy，去创建代理对象。Proxy 在 JDK 的反射包中。
- 对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用 CGLib（Code Generation Library）生成一个被代理对象的子类来作为代理。

![Image(12)](images/20190723215722974_32448.png)

## AOP 术语

### Advice 通知

通知定义了要织入目标对象的逻辑，以及执行时机。

Spring 中对应了 5 种不同类型的通知：

- 前置通知（Before）：在目标方法执行前，执行通知
- 后置通知（After）：在目标方法执行后，执行通知，此时不关系目标方法返回的结果是什么
- 返回通知（After-returning）：在目标方法执行后，执行通知
- 异常通知（After-throwing）：在目标方法抛出异常后执行通知
- 环绕通知（Around）: 目标方法被通知包裹，通知在目标方法执行前和执行后都被会调用

### Pointcut 切入点

如果说通知定义了在何时执行通知，那么切点就定义了在何处执行通知。

所以切点的作用就是通过匹配规则查找合适的连接点（Joinpoint），AOP 会在这些连接点上织入通知。

### Aspect 切面

切面包含了通知和切点，通知和切点共同定义了切面是什么，在何时，何处执行切面逻辑。

[Spring AOP 源码解析](https://www.javadoop.com/post/spring-aop-source?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)