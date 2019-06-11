### 基于 Java 的容器配置

#### 基础概念: @Bean 和 @Configuration
Spring 新的 Java 配置支持中的中心构件是 `@Configuration` 注解的类和 `@Bean` 注解的方法  
`@Bean` 注解用于指示方法实例化, 配置, 初始化由 Spring IoC 容器管理的新对象; 对于那些熟悉 Spring 的 `<beans/>` XML 配置的人来说, `@Bean` 注解扮演的角色与 `<bean/>` 元素相同; 你可以将 `@Bean` 带注释的方法与任何 Spring `@Component` 一起使用, 但是它们最常于与`@Configuration` bean 一起使用  
使用 `@Configuration` 注解类表示其主要目的是作为 bean 定义的源; 此外 `@Configuration` 类允许通过简单地调用同一个类中的其他 `@Bean` 方法来定义 bean 间依赖关系; 最简单的 `@Configuration` 类如下
```
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```
上面的 `AppConfig` 类将等效于以下 Spring `<beans/>` XML
```
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```
>完整的 `@Configuration` vs 精简的 `@Bean`
当 `@Bean` 方法在未使用 `@Configuration` 注解的类中声明时, 它们被称为以 "精简" 模式处理; 在 `@Component` 或甚至普通旧类中声明的 Bean方法将被视为 "精简的", 其中包含类的主要目的不同, 而 `@Bean` 方法只是一种奖励; 例如, 服务组件可以通过每个适用的组件类上的附加 `@Bean` 方法将管理视图公开给容器; 在这种情况下, `@Bean` 方法是一种简单的通用工厂方法机制
与完整的 `@Configuration` 不同, 精简的 `@Bean` 方法不能声明 bean 间依赖关系; 相反, 它们对其包含组件的内部状态以及它们可能声明的参数进行操作; 因此, 这样的 `@Bean` 方法不应该调用其他 `@Bean` 方法; 每个这样的方法实际上只是一个特定 bean 引用的工厂方法, 没有任何特殊的运行时语义; 这里的积极副作用是不必在运行时应用 CGLIB 子类, 因此在类设计方面没有限制 (即包含类可能是 final 的等等)
在常见的场景中, `@Bean` 方法将在 `@Configuration` 类中声明, 确保始终使用 "完整" 模式, 因此交叉方法引用将被重定向到容器的生命周期管理; 这将防止通过常规 Java 调用意外地调用相同的 `@Bean` 方法, 这有助于减少在 "精简" 模式下操作时难以跟踪的细微错误

`@Bean` 和 `@Configuration` 注解将在下面的部分中进行深入讨论; 首先, 我们将介绍使用基于 Java 的配置创建 Spring 容器的各种方法

#### 使用 AnnotationConfigApplicationContext 实例化 Spring 容器
下面的部分记录了 Spring 的 `AnnotationConfigApplicationContext`, Spring 3.0 中的新增内容; 这个多功能的 `ApplicationContext` 实现不仅能够接受 `@Configuration` 类作为输入, 还能接受使用 `JSR-330` 元数据注解的类和普通的 `@Component`类  
当 `@Configuration` 类作为输入提供时, `@Consfiguration` 类本身被注册为 bean定义, 并且类中所有声明的 `@Bean` 方法也被注册为 bean 定义  
当提供 `@Component` 和 `JSR-330` 类时, 它们被注册为 bean 定义, 并且假设在必要时在这些类中使用诸如 `@Autowired` 或 `@Inject` 之类的依赖注入元数据

##### 简单构造
与实例化 `ClassPathXmlApplicationContext` 时将 Spring `XML` 文件用作输入的方式大致相同, 在实例化 `AnnotationConfigApplicationContext` 时, `@Configuration` 配置类可用作输入; 这允许 Spring 容器的完全不使用 XML 配置
```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```
如上所述, `AnnotationConfigApplicationContext` 不仅限于使用 `@Configuration` 类; 任何 `@Component` 或 `JSR-330` 带注解的类都可以作为输入提供给构造函数; 例如
```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```
以上假设 `MyServiceImpl, Dependency1, Dependency2` 使用 Spring 注解依赖注入, 例如 `@Autowired`

##### 使用 `register(Class <?> ...)` 编程方式构建容器
可以使用无参构造函数实例化 `AnnotationConfigApplicationContext`, 然后使用 `register()` 方法进行配置; 在以编程方式构建 `AnnotationConfigApplicationContext` 时, 此方法特别有用
```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

##### 使用 `scan(String ...)` 启用组件扫描
要启用组件扫描, 只需注解 `@Configuration` 类, 如下所示
```
@Configuration
@ComponentScan(basePackages = "com.acme")
public class AppConfig  {
    ...
}
```
>有经验的 Spring 用户将熟悉与 Spring 的上下文相同的 XML 声明: namespace
>```
><beans>
>    <context:component-scan base-package="com.acme"/>
</beans>
>```
在上面的示例中, 将扫描 `com.acme` 包, 查找任何 `@Component-annotated` 类, 这些类将在容器中注册为 Spring bean 定义; `AnnotationConfigApplicationContext` 公开 `scan(String ...)`方法以允许相同的组件扫描功能
```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```
>请记住 `@Configuration` 类是使用 `@Component` 进行元注解的, 因此它们是组件扫描的候选者! 在上面的例子中, 假设 `AppConfig` 是在 `com.acme` 包 (或下面的任何包) 中声明的, 它将在调用 `scan()` 期间被拾取, 并且在 `refresh()` 时, 它的所有 `@Bean` 方法都将被处理并且在容器中注册为 bean 定义

##### 使用 `AnnotationConfigWebApplicationContext` 支持 web 应用
`AnnotationConfigApplicationContext` 的 `WebApplicationContext` 变体 `AnnotationConfigWebApplicationContext` 是一起提供的; 配置 Spring `ContextLoaderListener` servlet 侦听器, Spring MVC `DispatcherServlet` 等时可以使用此实现; 以下是配置典型 Spring MVC Web 应用程序的 `web.xml` 片段; 注意 `contextClass` context-param 和 init-param 的使用
```
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

#### 使用 `@Bean` 注解
`@Bean` 是方法级注释, 是 XML `<bean/>` 元素的直接模拟; 注释支持 `<bean/>` 提供的一些属性, 例如: `init-method, destroy-method, autowiring, name`  
你可以在 `@Configuration` 注解的类或 `@Component` 注解的类中使用 `@Bean` 注解

##### 声明一个 bean
要声明 bean, 只需使用 `@Bean` 注解的方法即可; 你可以使用此方法在指定为方法返回值的类型的 `ApplicationContext` 中注册 bean 定义; 默认情况下, bean 名称将与方法名称相同; 以下是 `@Bean` 方法声明的简单示例
```
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

>**参考:**  
[Java-based container configuration](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-java)
