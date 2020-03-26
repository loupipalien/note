### Spring 之旅

#### 简化 Java 开发
为了降低 Java 开发的复杂性, Spring 采取了以下 4 种关键策略
- 基于 POJO 的轻量级和最小侵入性编程
- 通过依赖注入和面向接口实现松耦合
- 基于切面和惯例进行声明式编程
- 通过切面和模板减少样板式代码

##### 激发 POJO 的潜能
TODO

##### 依赖注入
###### DI 功能是如何实现的
- 构造器注入

###### 装配
创建应用组件之间的协作的行为通常称为装配

###### 观察它如何工作
Spring 通过应用上下文 (Application Context) 装载 Bean 的定义并把它们组装起来, Spring 应用上下文全权负责对象的创建和组装; Spring 自带了多种应用上下文的实现, 它们之间主要的区别仅仅在于如何加载配置

##### 应用切面
DI 能够让相互协作的软件组织保持松散耦合, 而面向切面编程 (aspect-oriented programming, AOP) 允许把遍布应用各处的功能分离出来形成可重用的组件  
面向切面编程往往被定义为促使软件系统实现关注点的分离的一项技术, 系统由许多不通融的组件组成, 每一个组件各负责一块特定的功能, 除了实现自身核心的功能之外, 这些组件还经常承担着额外的功能; 诸如日志, 事务管理, 安全这样的系统服务经常融入到自身具有核心业务逻辑的组件中去, 这些系统服务通常被称为横切关注点, 因为它们会跨越系统的多个组件; 如果将这些关注点分散到多个组件中去, 代码将带来双重的复杂性
- 实现系统关注点功能的代码将会重复出现在多个组件中
- 组件会因为那些与自身核心代码无关的代码而变得混乱

##### 使用模板消除样板式代码
TODO

#### 容纳你的 Bean
Spring 容器并不是只有一个, Spring 自带了多个容器实现, 可以归为两种不同的类型
- Bean 工厂
由 `org.springframework.beans.factory.BeanFactory` 接口定义, 是最简单的容器, 提供基本的 DI 支持
- 应用上下文
由 `org.springframework.context.ApplicationContext` 接口定义, 基于 BeanFactory 构建, 并提供应用框架级别的服务, 例如从属性文件解析文本信息以及发布应用事件给感兴趣的时间监听者

##### 使用应用上下文
- AnnotationConfigApplicationContext
从一个或多个基于 Java 的配置类中加载 Spring 应用上下文
- AnnotationConfigWebApplicationContext
从一个或多个基于 Java 的配置类中加载 Spring Web 应用上下文
- ClassPathXmlApplicationContext
从类路径的一个或多个 XML 配置文件中加载上下文定义, 把应用上下文的定义文件作为类资源
- FileSystemXmlApplicationConext
从文件系统下的一个或多个 XML 配置文件中加载上下文定义
- XmlWebApplicationContext
从 Web 应用下的一个或多个 XML 配置文件中加载上下文定义

##### Bean 的生命周期
![Bean 在 Spring 容器中的生命周期](http://static2.iocoder.cn/images/Spring/2018-12-24/03.jpg)  
- Spring 对 Bean 进行实例化
- Spring 将值和 Bean 的引用注入到 Bean 对应的属性中
- 如果 Bean 实现了 `BeanNameAware` 接口, Spring 将 Bean 的 ID 传递给 `setBeanName()` 方法
- 如果 Bean 实现了 `BeanFactoryAware` 接口, Spring 将调用 `setBeanFactory()` 方法, 将 `BeanFactory` 容器实例传入
- 如果 Bean 实现了 `ApplicationContextAware` 接口, Spring 将调用 `setApplicationContext()` 方法将 Bean 所在的应用上下文传入
- 如果 Bean 实现了 `BeanPostProcessor` 接口, Spring 将调用它们的 `postProcessBeforeInitialization()` 方法
- 如果 Bean 实现了 `InitializingBean` 接口, Spring 将调用它们的 `afterPropertiesSet()` 方法
- 如果 Bean 使用了 `init-method` 声明了初始化方法, 该方法也会被调用
- 如果 Bean 实现了 `BeanPostProcessor` 接口, Spring 将调用它们的 `postProcessAfterInitialization()` 方法
- 此时 Bean 已经准备就绪, 可以被应用程序使用了, 它们将一直驻留在应用上下文中, 直到该应用上下文被销毁
- 如果 Bean 实现了 `DisposableBean` 接口, Spring 将调用它们的 `destory()` 方法
- 如果 Bean 使用了 `destory-method` 声明了初始化方法, 该方法也会被调用

#### 俯瞰 Spring 风景线
##### Spring 模块
![Spring 模块](http://static2.iocoder.cn/images/Spring/2018-12-24/01.jpg)

###### Spring 核心容器
###### Spring 的 AOP 模块
###### 数据访问与集成
###### Web 与远程调用
###### Instrumentation
###### 测试

##### Spring Portfilio
TODO

#### Spring 的新功能
TODO
