### BeanFactory
`BeanFactory` API 为 Spring 的 IoC 功能提供了基础; 它的特定契约主要用于与 Spring 的其他部分和相关的第三方框架集成, 其 `DefaultListableBeanFactory` 实现是更高级别 `GenericApplicationContext` 容器中的密钥委托  
`BeanFactory` 和相关接口 (如 `BeanFactoryAware, InitializingBean, DisposableBean`) 是其他框架组件的重要集成点: 不需要任何注释甚至反射, 它们允许容器与其组件之间进行非常有效的交互; 应用程序级 bean 可以使用相同的回调接口, 但通常更喜欢通过注解或通过编程配置进行声明性依赖注入  
请注意, 核心 `BeanFactory` API 级别及其 `DefaultListableBeanFactory` 实现不会对配置格式或要使用的任何组件注解做出假设; 所有这些风格都通过扩展来实现, 例如 `XmlBeanDefinitionReader` 和 `AutowiredAnnotationBeanPostProcessor`, 它们在共享 `BeanDefinition` 对象上作为核心元数据表示进行操作; 这是使 Spring 的容器如此灵活和可扩展的本质  
以下部分解释了 `BeanFactory` 和 `ApplicationContext` 容器级别之间的差异以及对引导的影响

#### BeanFactory 或者 ApplicationContext
使用 `ApplicationContext`, 除非你有充分的理由不这样做, 使用 `GenericApplicationContext` 及其子类 `AnnotationConfigApplicationContext` 作为自定义引导的常见实现; 这些是 Spring 用于所有常见目的的核心容器的主要入口点: 加载配置文件, 触发类路径扫描, 以编程方式注册 bean 定义和带注解的类  
因为 `ApplicationContext` 包含 `BeanFactory` 的所有功能, 所以通常建议使用简单的 `BeanFactory`, 除了需要完全控制 bean 处理的场景; 在诸如 `GenericApplicationContext` 实现之类的 `ApplicationContext` 中, 将按惯例 (即通过 bean 名称或 bean 类型) 检测几种 bean, 特别是 post-processors, 而普通的 `DefaultListableBeanFactory` 对任何特殊 bean 都是不可知的  
对于许多扩展容器功能, 例如注解处理和AOP代理, `BeanPostProcessor` 扩展点是必不可少的; 如果仅使用普通的 `DefaultListableBeanFactory`, 则默认情况下不会检测到并激活此类 post-processors; 这种情况可能令人困惑, 因为你的 bean 配置实际上没有任何问题, 它更像是在这种情况下需要通过附加设置完全自举的容器  
下表列出了 `BeanFactory` 和 `ApplicationContext` 接口和实现提供的功能

| 功能 | BeanFactory | ApplicationContext |
| :--- | :--- |:--- |
| Bean 的初始化和写入 | YES | YES |
| 整合生命周期管理 | NO | YES |
| 自动 `BeanPostProcessor` 注册 | NO | YES |
| 自动 `BeanFactoryPostProcessor ` 注册 | NO | YES |
| 方便的 `MessageSource` 访问 (为了初始化) | NO | YES |
| 内置的 `ApplicationEvent` 公布机制 | NO | YES |

要使用 `DefaultListableBeanFactory` 显式注册 bean post-processors, 你需要以编程方式调用 `addBeanPostProcessor`
```
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
factory.addBeanPostProcessor(new MyBeanPostProcessor());

// now start using the factory
```
要将 `BeanFactoryPostProcessor` 应用于普通的 `DefaultListableBeanFactory`, 需要调用其 `postProcessBeanFactory` 方法
```
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// bring in some property values from a Properties file
PropertyPlaceholderConfigurer cfg = new PropertyPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// now actually do the replacement
cfg.postProcessBeanFactory(factory);
```
在这两种情况下, 显式注册步骤都不方便, 这就是为什么各种 `ApplicationContext` 变体优先于 Spring 支持的应用程序中的普通 `DefaultListableBeanFactory`, 特别是在典型的企业设置中依赖 `BeanFactoryPostProcessors` 和 `BeanPostProcessors` 来扩展容器功能的原因
>`AnnotationConfigApplicationContext` 具有开箱即用的所有通用注解 post-processors, 并且可以通过诸如 `@EnableTransactionManagement` 之类的配置注解在封面下引入额外的处理器; 在 Spring 基于注解的配置模型的抽象级别, bean post-processors 的概念变成仅仅是内部容器细节

#### 胶水代码和邪恶的单例
TODO

>**参考:**  
[The BeanFactory](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-beanfactory)
