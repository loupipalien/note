### Spring 框架介绍
Spring 框架是一个为开发 Java 应用提供全面基础支持的 Java 平台; Spring 处理基础架构, 这样你可以专注你的应用程序  
Spring 使你可以从 plain old Java objects (POJOs) 构建应用程序, 并且可以无侵入的将企业服务应用于 POJOs;
这些功能可以应用于 Java SE 编程模型以及全部或部分的 Java EE  
作为一名应用程序开发者, 你可以从 Spring 平台收益示例如下:
- 使一个在数据库事务中执行的 Java 方法不用处理事务的 APIs
- 使一个本地 Java 方法的 HTTP 端不用处理 Servlet API
- 使一个本地 Java 方法的消息处理器不用处理 JMS API
- 使一个本地 Java 方法的管理操作不用处理 JMX API

#### 依赖注入和控制反转
Java应用程序 - 从受约束的嵌入式应用程序到n层服务器端企业应用程序的宽松术语 - 通常由协作形成应用程序的对象组成; 因此, 应用程序中的对象彼此依赖  
虽然 Java 平台提供了丰富的应用程序开发功能, 但是它缺少将基本构建块组织成连贯整体的方法, 而将这项任务留给了架构师和开发者; 虽然你可以使用例如工厂, 抽象工厂, 建造者, 装饰器, 服务定位器等设计模式组合各种类和对象实例来组成一个应用程序, 这些模式简单来说是: 给予最佳实践的一个名称, 描述这些模式的作用, 它应用的位置, 它解决的问题, 等等; 模式是在你的应用程序中你必须实现的规范化的最佳实践  
Spring 框架控制反转 (IoC) 组件解决这个问题是通过提供一种将不同组件组合成一个完全可以使用的应用程序的规范化方法; Spring 框架将编写规范化的设计模式作为可以集成到你自己的应用程序中的头等对象; 许多组织和机构以这种方式使用 Spring 框架来构建健壮的, 可维护的应用程序
>**背景**
"问题是, [它们] 是控制哪个方面的反转?"; 2004 年, Martin Fowler 在 [自己的网站](http://martinfowler.com/articles/injection.html) 上提出的关于控制反转的这个问题; Fowler 建议重命名这个原则使它更具有自解释性并且提出了注入依赖

#### 框架模块
Spring 框架由大约由 20 多个模块的功能组成; 这些模块分为 Core Container, Data Access/Integration, Web, AOP (Aspect Oriented Programming), Instrumentation, Messaging, and Test, 如下图所示
![Overview of the Spring Framework](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/images/spring-overview.png)
以下章节列出了每个功能的可用模块及其构件名称及其涵盖的主题; 构件名称与依赖管理工具中的构建 ID 相关联

##### 核心容器
[核心容器](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-introduction) 由 `spring-core`, `spring-beans`, `spring-context`, `spring-context-support`, 和 `spring-expression` (Spring Expression Language) 模块组成  
`spring-core` 和 `spring-beans` 模块提供了 [框架的基础部分](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-introduction), 包括 IoC 和依赖注入功能; `BeanFactory` 是工厂模式的一个复杂实现; 它消除了对编程单例的需求, 并允许你从你的实际程序逻辑中分离出依赖的配置的规范  
[Context](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#context-introduction) (`spring-context`) 模块构建于 [Core 和 Beans](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-introduction) 模块提供的稳固的基础之上: 它是一个类似与 JNDI 注册的使用框架风格形式访问对象的方法; Context 模块从 Beans 模块继承了它的功能并添加了对国际化的支持 (例如, 使用资源绑定), 事件传播, 资源加载, 以及透明的上下文创建, 例如 Servlet 容器; Context 模块也支持例如 EJB, JMX, 以及基本的远程等 Java EE 功能; `ApplicationContext` 接口是 Context 模块的重点; `spring-context-support` 提供了对通用的第三方库集成到 Spring 应用程序上下文的中支持, 以进行缓存 (EhCache, Guava, JCache), 邮件 (JavaMail), 调度(CommonJ, Quartz) 以及模板引擎 (FreeMarker, JasperReports, Velocity)  
`spring-expression` 模块提供了一个强大的 [Expression Language](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/expressions.html), 用于在运行时查询和操纵对象图; 它是 JSP 2.1 规范中指定的统一表达式语言 (统一的 EL) 的扩; 此语言支持属性值的设置与获取, 属性分配, 方法调用, 数组, 集合, 索引内容的访问, 逻辑和算术操作符, 命名变量, 从 Spring IoC 容器中按名称搜索对象; 它还支持列表投影和选择以及常用的列表聚合

##### AOP 和工具
`spring-aop` 模块提供了 [AOP](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/aop.html#aop-introduction) Alliance 兼容的面向切面编程的实现, 允许你定义, 例如方法拦截器和切入点, 以干净地解耦实现应该分离的功能的代码; 使用源码级别的元数据功能, 你可以将行为信息合并到你的代码中, 以一种类似于 .NET 属性的方式  
单独的模块 `spring-aspects` 提供了与 AspectJ 的整合  
`spring-instrument` 模块提供了用于特定应用服务器的类工具支持和类加载器实现; `spring-instrument-tomcat` 模块包含对 Tomcat 的 Spring 工具代理

##### Messaging
