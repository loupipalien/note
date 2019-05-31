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
>虽然 `BeanPostProcessor` 注册的推荐方法是通过 `ApplicationContext` 自动检测 (如上所述), 但也可以使用 `addBeanPostProcessor` 方法以编程方式对 `ConfigurableBeanFactory` 注册它们; 当需要在注册之前评估条件逻辑, 或者甚至在层次结构中跨上下文复制 bean post-processor 时, 这非常有用; 但请注意, 以编程方式添加的 `BeanPostProcessors` 不遵循 `Ordered` 接口; 而是注册的顺序决定了执行的顺序; 另请注意, 无论是否有任何显式排序, 以编程方式注册的 `BeanPostProcessors` 始终在通过自动检测注册的 `BeanPostProcessors` 之前处理
>实现 `BeanPostProcessor` 接口的类是特殊的, 容器会对它们进行不同的处理; 直接引用的所有 `BeanPostProcessors` 和 bean 都会在启动时实例化, 这是 `ApplicationContext` 的特殊启动阶段的一部分; 接下来, 所有 `BeanPostProcessors` 都以排序方式注册, 并应用于容器中的所有其他 bean; 因为 AOP 自动代理是作为 `BeanPostProcessor` 本身实现的, 所以 `BeanPostProcessors` 和它们直接引用的 bean 都不适合自动代理, 因此不会有切面织入它们
>对于任何此类 bean, 你应该看到一条信息性日志消息： "Bean foo 不适合所有 `BeanPostProcessor` 接口处理 (例如: 不符合自动代理条件)"
>请注意, 如果使用自动装配或 `@Resource` (可能会回退到自动装配) 将 Bean 连接到 `BeanPostProcessor`, 则 Spring 可能会在搜索类型匹配依赖项候选项时访问意外的 bean, 从而使它们不符合自动代理或其他类型的 bean post-processor; 例如, 如果你有一个使用 `@Resource` 注释的依赖项, 其中 field/setter 名称不直接对应于 bean 的声明名称而且没有使用 name 属性, 那么 Spring 将访问其他 bean 以按类型匹配它们

以下示例显示如何在 `ApplicationContext` 中编写, 注册和使用 `BeanPostProcessors`

##### 示例: Hello World, BeanPostProcessor-style
第一个例子说明了基本用法; 该示例显示了一个自定义 `BeanPostProcessor` 实现, 该实现在容器创建时调用每个 bean 的 `toString()` 方法, 并将生成的字符串输出到系统控制台  
在下面找到自定义 `BeanPostProcessor` 实现类定义:
```
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/lang
        https://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"
            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```



>**参考:**
[Container Extension Points](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-extension)
