### 容器扩展点
通常, 应用程序开发人员不需要继承 `ApplicationContext` 实现类; 然而, 可以通过插入特殊集成接口的实现来扩展 Spring IoC 容器;  接下来的几节将介绍这些集成接口

#### 使用 BeanPostProcessor 自定义 beans
`BeanPostProcessor` 接口可以定义你实现的回调方法, 以提供你自己的 (或覆盖容器的默认) 实例化逻辑, 依赖关系解析逻辑等; 如果要在 Spring 容器完成实例化, 配置和初始化 bean 之后实现某些自定义逻辑, 则可以插入一个或多个自定义 `BeanPostProcessor` 实现   
你可以配置多个 `BeanPostProcessor` 实例, 并且可以通过设置 `order` 属性来控制这些 `BeanPostProcessors` 的执行顺序; 仅当 `BeanPostProcessor` 实现 `Ordered` 接口时, 才能设置此属性; 如果你编写自己的 `BeanPostProcessor`, 你也应该考虑实现 `Ordered` 接口; 有关更多详细信息, 请参阅 `BeanPostProcessor` 和 `Ordered` 接口的 javadoc; 另请参阅下面有关 [BeanPostProcessors 编程注册](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-programmatically-registering-beanpostprocessors) 的说明
>`BeanPostProcessors` 在 bean (或对象) 实例上运行; 也就是说, Spring IoC 容器实例化一个 bean 实例, 然后 `BeanPostProcessors` 完成它们的工作
>`BeanPostProcessors` 的范围是每个容器; 这仅在你使用容器层次结构时才有意义; 如果在一个容器中定义 `BeanPostProcessor`, 它将只对该容器中的 bean 进行后处理; 换句话说, 在一个容器中定义的 bean 不会被另一个容器中定义的 `BeanPostProcessor` 进行后处理, 即使两个容器都是同一层次结构的一部分
>要更改实际的 bean 定义 (即定义 bean 的蓝图), 你需要使用 `BeanFactoryPostProcessor`, 见 [第 7.8.2 节 "使用 BeanFactoryPostProcessor 定制配置元数据"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-extension-factory-postprocessors)

`org.springframework.beans.factory.config.BeanPostProcessor` 接口由两个回调方法组成; 当这样的类被注册为带容器的后处理器时, 对于容器创建的每个 bean 实例, post-processor 在 bean 初始化方法之前 (例如 `InitializingBean` 的 `afterPropertiesSet()` 或任何声明的 init 方法) 以及 bean 初始化方法之后都会调用; post-processor 可以对 bean 实例执行任何操作, 包括完全忽略回调; bean post-processor 通常会检查回调接口, 或者可以使用代理包装 bean; 一些 Spring AOP 基础结构类实现为 bean post-processor, 以便提供代理包装逻辑  
`ApplicationContext` 自动检测定义在配置元数据中实现了 `BeanPostProcessor` 接口的 bean; `ApplicationContext` 将这些 bean 注册为 post-processor, 以便稍后在创建 bean 时调用它们; Bean post-processor 可以像任何其他 bean 一样部署在容器中  
请注意, 在配置类上使用 `@Bean` 工厂方法声明 `BeanPostProcessor` 时, 工厂方法的返回类型应该是实现类本身, 或者至少是 `org.springframework.beans.factory.config.BeanPostProcessor` 接口, 清楚地表明该 bean 的 post-processor 性质; 否则, `ApplicationContext` 将无法在完全创建之前按类型自动检测它; 由于 `BeanPostProcessor` 需要尽早实例化以便应用于上下文中其他 bean 的初始化, 因此这种早期类型检测至关重要


>**参考:**
[Container Extension Points](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-extension)
