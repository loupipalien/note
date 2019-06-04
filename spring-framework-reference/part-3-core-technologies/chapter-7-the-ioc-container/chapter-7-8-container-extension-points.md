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
请注意如何简单地定义 `InstantiationTracingBeanPostProcessor`; 它甚至可以没有名称, 因为它是一个 bean, 它可以像任何其他 bean 一样依赖注入; (前面的配置还定义了一个由 `Groovy` 脚本支持的 bean, Spring 动态语言支持在 [第 35 章动态语言支持](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/dynamic-language.html) 的章节中有详细介绍)  
以下简单 Java 应用程序执行上述代码和配置
```
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = (Messenger) ctx.getBean("messenger");
        System.out.println(messenger);
    }

}
```
上述应用程序的输出类似于以下内容
```
Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961
org.springframework.scripting.groovy.GroovyMessenger@272961
```

##### 示例: RequiredAnnotationBeanPostProcessor
将回调接口或注解与自定义 `BeanPostProcessor` 实现结合使用是扩展 Spring IoC 容器的常用方法; 一个例子是 Spring 的 `RequiredAnnotationBeanPostProcessor` ---  一个随 Spring 发行版一起提供的 `BeanPostProcessor` 实现, 它确保用 (任意) 在 beans 上注解标记的 JavaBean 属性实际上 (配置为) 依赖注入值

#### 使用 BeanFactoryPostProcessor 自定义配置元数据
我们将看到的下一个扩展点是 `org.springframework.beans.factory.config.BeanFactoryPostProcessor`; 这个接口的语义类似于 `BeanPostProcessor` 的语义, 主要区别在于: `BeanFactoryPostProcessor` 对 bean 配置元数据进行操作; 也就是说, Spring IoC 容器允许 `BeanFactoryPostProcessor` 读取配置元数据, 并可能在容器实例化除 `BeanFactoryPostProcessor` 之外的任何 bean 之前更改它  
你可以配置多个 `BeanFactoryPostProcessor`, 并且可以通过设置 `order` 属性来控制这些 `BeanFactoryPostProcessor` 的执行顺序; 然而只能在 `BeanFactoryPostProcessor` 实现 `Ordered` 接口设置此属性; 如果编写自己的 `BeanFactoryPostProcessor`, 则应考虑实现 `Ordered` 接口; 有关更多详细信息, 见 `BeanFactoryPostProcessor` 和 `Ordered` 接口的 javadoc
>如果要更改实际的 bean 实例 (即从配置元数据创建的对象), 则需要使用 `BeanPostProcessor` (如上面 [第 7.8.1 节 "使用 BeanPostProcessor 定制 bean"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-extension-bpp) 中所述); 虽然技术上可以在 `BeanFactoryPostProcessor` 中使用 bean 实例 (例如, 使用 `BeanFactory.getBean()`), 但这样做会导致 bean 的过早实例化， 从而违反标准的容器生命周期; 这可能会导致负面影响, 例如绕过 bean post-processor
>此外, `BeanFactoryPostProcessors` 的范围是每个容器; 这仅在你使用容器层次结构时才有意义; 如果在一个容器中定义 `BeanFactoryPostProcessor`, 它将仅应用于该容器中的 bean 定义; `BeanFactoryPostProcessor` 不会在另一个容器中对一个容器中的 Bean 定义进行后处理, 即使两个容器都是同一层次结构的一部分

bean factory post-processor 在 `ApplicationContext` 中声明时自动执行, 以便将更改应用于定义容器的配置元数据; Spring 包含许多预定义的 bean 工厂后处理器, 例如 `PropertyOverrideConfigurer` 和 `PropertyPlaceholderConfigurer`; 例如, 也可以使用自定义 `BeanFactoryPostProcessor` 来注册自定义属性编辑器  
`ApplicationContext` 自动检测部署到其中的任何实现 `BeanFactoryPostProcessor` 接口的 bean; 它在适当的时候使用这些 bean 作为 bean factory post-processor; 你可以像处理任何其他 bean 一样部署这些 post-processor bean  
>与 `BeanPostProcessor` 一样, 你通常不希望为 `BeanFactoryPostProcessor` 配置延迟初始化; 因为如果没有其他 bean 引用 `Bean (Factory) PostProcessor`, 则该后处理器根本不会被实例化; 因此, 其标记为延迟初始化将被忽略, 即使在 `<beans/>` 元素的声明中将 `default-lazy-init` 属性设置为 true, 也会急切地实例化 Bean (Factory) PostProcessor

##### 示例: 类名替换 PropertyPlaceholderConfigurer
使用 `PropertyPlaceholderConfigurer` 可以将 bean 定义中的属性值外部化在一个使用标准 Java Properties 格式的单独文件中; 这样做可以使部署应用程序的人员自定义特定于环境的属性 (如数据库 URL 和密码), 而不会出现修改主 XML 定义文件或容器文件的复杂性或风险  
请考虑以下基于 XML 的配置元数据片段, 其中定义了具有占位符值的 `DataSource`; 该示例显示了从外部属性文件配置的属性; 在运行时, `PropertyPlaceholderConfigurer` 应用于将替换 `DataSource` 的某些属性的元数据; 要替换的值指定为 `${property-name}` 形式的占位符, 该形式遵循 `Ant/log4j/JSPEL` 样式
```
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations" value="classpath:com/foo/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```
实际值来自标准 Java Properties 格式的另一个文件
```
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```
因此, 字符串 `${jdbc.username}` 在运行时将替换为值 'sa', 这同样适用于与属性文件中的键匹配的其他占位符值; `PropertyPlaceholderConfigurer` 检查 bean 定义的大多数属性和属性中的占位符; 此外, 可以自定义占位符前缀和后缀  
使用 Spring 2.5 中引入的上下文命名空间, 可以使用专用配置元素配置属性占位符; 可以在 `location` 属性中提供一个或多个逗号分隔的列表作为 locations
```
<context:property-placeholder location="classpath:com/foo/jdbc.properties"/>
```
`PropertyPlaceholderConfigurer` 不仅在你指定的属性文件中查找属性; 默认情况下, 如果它在指定的属性文件中找不到属性, 它还会检查 Java System 属性; 你可以通过使用以下三个受支持的整数值之一设置 configurer 的 `systemPropertiesMode` 属性来自定义此行为
- never(0): 不检查系统属性
- fallback(1): 如果在指定的属性文件中无法解析, 则检查系统属性; 这是默认值
- override(2): 在尝试指定的属性文件之前, 首先检查系统属性; 这允许系统属性覆盖任何其他属性源

更多信息请参阅 `PropertyPlaceholderConfigurer` 的 javadocs
>你可以使用 `PropertyPlaceholderConfigurer` 替换类名, 这在你必须在运行时选择特定实现类时有时很有用; 例如
>```
><bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
>    <property name="locations">
>        <value>classpath:com/foo/strategy.properties</value>
>    </property>
>    <property name="properties">
>        <value>custom.strategy.class=com.foo.DefaultStrategy</value>
>    </property>
></bean>
>
><bean id="serviceStrategy" class="${custom.strategy.class}"/>
>```
>如果在运行时无法将类解析为有效类, 则在即将创建 bean 时将失败, 当非 lazy-init bean的 `ApplicationContext` 的 `preInstantiateSingletons()` 阶段期间实例化时

##### 示例: PropertyOverrideConfigurer
`PropertyOverrideConfigurer` 是另一个bean factory post-processor, 类似于 `PropertyPlaceholderConfigurer`, 但与后者不同, 原始定义可以具有默认值, 或者根本不具有 bean 属性的值; 如果重写的 Properties 文件没有某个 bean 属性的条目, 则使用默认的上下文定义  
请注意, bean 定义不知道被覆盖, 因此不能够立刻知道 XML 定义文件中使用覆盖配置器; 如果多个 `PropertyOverrideConfigurer` 实例为同一个 bean 属性定义了不同的值, 则由于覆盖机制, 最后一个实例将生效  
Properties 文件配置格式如下
```
beanName.property=value
```
例如
```
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql:mydb
```
此示例文件可以与包含名为 dataSource 的 bean 的容器定义一起使用, 该 bean 具有 driver 和 url 属
性  
也支持复合属性名称, 只要路径的每个组件 (重写的最终属性除外) 都已经非空 (可能由构造函数初始化); 在这个例子中......
```
foo.fred.bob.sammy=123
```
将 foo bean 的 fred 属性的 bob 属性的 sammy 属性设置为 123
>指定的覆盖值始终是字面值; 它们不会被翻译成 bean引用; 当 XML bean 定义中的原始值指定 bean 引用时, 此约定也适用

使用 Spring 2.5 中引入的上下文命名空间, 可以使用专用配置元素配置属性覆盖
```
<context:property-override location="classpath:override.properties"/>
```

#### 使用 FactoryBean 自定义实例化逻辑
为本身为工厂的对象实现 `org.springframework.beans.factory.FactoryBean` 接口  
`FactoryBean` 接口是 Spring IoC 容器实例化逻辑的可插拔点; 如果你有更复杂的初始化代码, 这些代码在 Java 中更好地表达, 而不是 (可能) 冗长的 XML, 你可以创建自己的 `FactoryBean`, 在该类中编写复杂的初始化, 然后将自定义 `FactoryBean` 插入容器中  
`FactoryBean` 接口提供以下三个方法
- `Object getObject()`: 返回此工厂创建的对象的实例; 可以共享实例, 具体取决于此工厂是返回单例还是原型
- `boolean isSingleton()`: 如果此 FactoryBean 返回单例, 则返回 `true`, 否则返回 `false`
- `Class getObjectType()`: 返回 `getObject()` 方法返回的对象类型, 如果事先不知道类型, 则返回 `null`

`FactoryBean` 概念和接口在 Spring Framework 中的许多地方使用; `FactoryBean` 接口的 50 多个实现随 Spring 一起提供  
当你需要向容器询问实际的 `FactoryBean` 实例本身而不是它生成的 bean 时, 在调用 `ApplicationContext` 的 `getBean()` 方法时, 使用 `&` 符号作为 bean 的 `id` 前缀; 因此, 对于 `id` 为 myBean 的给定 FactoryBean, 在容器上调用 `getBean("myBean")` 将返回 `FactoryBean` 的产生的 bean; 而调用 `getBean("&myBean")` 会返回 `FactoryBean` 实例本身

>**参考:**
[Container Extension Points](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-extension)
