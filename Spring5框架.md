# Spring 框架概述

Spring 是轻量级的开源的 JavaEE 框架

Spring 可以解决企业应用开发的复杂性

Spring 有两个核心部分：IOC 和 Aop

(1)IOC:控制反转，把创建对象过程交给Spring进行管理
(2)Aop:面向切面，不修改源代码进行功能增强
4、Spring特点
(1)方便解耦，简化开发
(2)Aop 编程支持
(3)方便程序测试
(4)方便和其他框架进行整合
(5)方便进行事务操作
(6)降低 API 开发难度

## 入门案例

spring5 需要的相关jar包：

- Beans
- Core
- Context
- Expression



# IOC 接口

**IOC 容器的底层就是对象工厂**



Spring提供两种IOC容器实现的方法：

1. `BeanFactory`：IOC 容器基本实现方式，是 Spring 内部的使用接口，不提供开发人员使用

   **加载配置文件的时候不会创建对象，在获取对象（使用）的时候才去创建对象**



2. `ApplicationContext`：`BeanFactory`的子接口，提供更多更强大的额功能，一般开发人员使用

   **加载配置文件的时候就会把配置文件对象进行创建**

如：在 tomcat 服务器启动的时候，把配置文件中的对象全部创建，直接使用，提高用户体验





## IOC 操作 bean 管理

### bean 管理是两个操作：

1. Spring 创建对象
2. Spring 注入属性



### bean 管理操作的两种方式

1. 基于 xml 文件
2. 基于注解方式实现



具体：

1. 基于 xml 创建对象

   **注意：创建对象默认使用无参构造器创建对象**

2. 基于 xml 注入属性



DI（依赖注入）：就是注入属性

1. 使用 set 方式注入
2. 使用有参构造器注入





# AOP面向切面

Spring AOP基于动态代理实现，

1. 如果要代理的对象**实现了某个接口**，那么Spring AOP就会使用 `JDK` 动态代理去创建代理对象；
2. 没有实现接口的对象，无法使用JDK动态代理，转而使用 `CGlib` 动态代理**生成一个被代理对象的子类**来作为代理对象。



著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/interview/x-interview-2.html

当然也可以使用`AspectJ`，Spring AOP中已经集成了AspectJ，AspectJ应该算得上是Java生态系统中最完整的AOP框架了。使用AOP之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样可以大大简化代码量。我们需要增加新功能也方便，提高了系统的扩展性。**日志功能**、**事务管理**和**权限管理**等场景都用到了AOP。