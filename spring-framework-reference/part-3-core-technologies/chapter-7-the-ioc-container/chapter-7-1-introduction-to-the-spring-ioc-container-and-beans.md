### Spring IoC 容器和 beans 介绍
本章涵盖了 Spring Framework 的控制反转实现 (IoC) 原理; IoC 也被称为依赖注入 (DI); 这是一个定义它们依赖的过程, 即它们使用的其他对象, 仅通过构造函数参数, 工厂方法的参数, 或者在构造或从工厂方法返回后在对象实例上设置的属性; 然后容器在创建 bean 时注入这些依赖; 这个过程基本上是反向的, 因此命名为控制反转 (IoC), bean 本身通过直接使用类构建器或者类似 Service Locator 模式的机制来控制依赖的实例化和位置  
`org.springframework.beans` 和 `org.springframework.context` 包是 Spring Framework IoC 容器的基础; `BeanFactory` 接口提供了能够管理任何类型对象的高级可配置机制; `ApplicationContext` 是 `BeanFactory` 的一个子接口; 它添加了更易与 Spring AOP 功能的集成; 消息资源处理 (为了用于国际化), 事件发布; 以及例如用于 web 应用的 `WebApplicationContext` 的应用层指定上下文  
简而言之, ·`BeanFactory` 提供了配置框架和基础功能, `ApplicationContext` 添加了更多企业级的功能; `ApplicationContext` 是 `BeanFactory` 的一个完整超集, 在本章中仅用于 Spring IoC 容器的描述; 更多关于使用 `BeanFactory` 而不是 `ApplicationContext` 的信息, 参见 [7.16 小节, BeanFactory](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-beanfactory)  
在 Spring 中, 构成你应用程序的主干的并且被 Spring Ioc 容器管理的对象称为 beans; 一个 bean 是一个被实例化的, 装备过的, 并且被 Sprign IoC 容器管理的对象, bean 是你应用程序中众多对象中的一个; Bean 及其之间的依赖关系反映在容器使用的配置元数据中

>**参考:**
[Introduction to the Spring IoC container and beans](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-introduction)
