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
上述配置与以下 Spring XML 完全等效
```
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```
这两个声明都在 `ApplicationContext` 中创建一个名为 `transferService` 的 bean, 绑定到 `TransferServiceImpl` 类型的对象实例
```
transferService -> com.acme.TransferServiceImpl
```
你还可以使用接口 (或基类) 声明你的 `@Bean` 方法的返回类型
```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```
但是, 这会将高级类型预测的可见性限制为指定的接口类型 (`TransferService`), 然后只有在实例化受影响的单例 bean 后, 容器才知道完整类型 (`TransferServiceImpl`); 非延迟单例 bean 根据其声明顺序进行实例化, 因此你可能会看到不同的类型匹配结果, 具体取决于另一个组件何时尝试通过非声明类型进行匹配 (例如 `@Autowired TransferServiceImpl` 在 "transferService" bean 被实例化后才会解析)
>如果你始终通过声明的服务接口引用你的类型, 则 `@Bean` 返回类型可以安全地加入该设计决策; 但是, 对于实现多个接口的组件或可能由其实现类型引用的组件, 更安全地声明可能的是最具体的返回类型 (至少与引用你的 bean 的注入点所需的具体相同)

##### Bean 依赖
`@Bean` 注解方法可以有任意数量的参数来描述构建该 bean 所需的依赖关系; 例如, 如果我们的 `TransferService` 需要 `AccountRepository`, 我们可以通过方法参数实现该依赖关系
```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```
解析机制与基于构造函数的依赖注入非常相似, 有关详细信息请参阅相关部分;

##### 接受的生命周期回调
使用 `@Bean` 注解定义的任何类都支持常规生命周期回调, 并且可以使用 `JSR-250` 中的 `@PostConstruct` 和 `@PreDestroy` 注解, 有关详细信息请参阅 [JSR-250 注解](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-postconstruct-and-predestroy-annotations)  
完全支持常规的 Spring 生命周期回调; 如果 bean 实现 `InitializingBean`, `DisposableBean` 或 `Lifecycle`, 则容器会调用它们各自的方法  
还完全支持标准的 `*Aware` 接口集, 例如 `BeanFactoryAware, BeanNameAware, MessageSourceAware, ApplicationContextAware` 等  
`@Bean` 注解支持指定任意初始化和销毁回调方法, 就像 bean 元素上的 Spring XML 的 `init-method` 和 `destroy-method` 属性一样
```
public class Foo {

    public void init() {
        // initialization logic
    }
}

public class Bar {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public Foo foo() {
        return new Foo();
    }

    @Bean(destroyMethod = "cleanup")
    public Bar bar() {
        return new Bar();
    }
}
```
>默认情况下, 使用 Java 配置定义的具有公共 `close` 或 `shutdown` 方法的 bean 会自动使用销毁回调登记; 如果你有一个公共 `close` 或 `shutdown` 方法, 并且你不希望在容器关闭时调用它, 只需将 `@Bean(destroyMethod = "")` 添加到你的 bean 定义中以禁用默认 (推断) 模式  
对于通过 JNDI 获取的资源, 你可能希望默认执行此操作, 因为其生命周期在应用程序外部进行管理; 特别是, 请确保始终为 `DataSource` 执行此操作, 因为已知它在 Java EE 应用程序服务器上存在问题
>```
>@Bean(destroyMethod="")
>public DataSource dataSource() throws NamingException {
>    return (DataSource) jndiTemplate.lookup("MyDS");
>}
>```
>此外, 使用 `@Bean` 方法, 你通常会选择使用编程 JNDI 查找: 使用 Spring 的 `JndiTemplate/JndiLocatorDelegate` 帮助程序或直接 JNDI `InitialContext` 用法, 但不使用 `JndiObjectFactoryBean` 变量, 这会强制你将返回类型声明为 `FactoryBean` 类型而不是实际的目标类型, 使其更难用于其他打算在此处引用所提供资源的 `@Bean` 方法中的交叉引用调用

当然, 在上面的 `Foo` 的情况下, 在构造期间直接调用 `init()` 方法同样有效
```
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        Foo foo = new Foo();
        foo.init();
        return foo;
    }

    // ...
}
```
>当你直接使用 Java 工作时, 你可以使用对象执行任何你喜欢的操作， 并且不必总是依赖于容器生命周期

##### 指定 bean 范围
##### 使用 @Scope 注解
你可以使用 `@Bean` 注解指定定义的 bean 应具有特定范围; 你可以使用 Bean Scopes 部分中指定的任何标准作用域  
默认范围是单例, 但你可以使用 `@Scope` 注解覆盖它
```
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```
##### @Scope 和范围代理
Spring 提供了一种通过作用 [范围代理](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-other-injection) 处理作用范围依赖项的便捷方法; 使用 XML 配置时创建此类代理的最简单方法是 `<aop:scoped-proxy/>` 元素; 使用 `@Scope` 注解在 Java 中配置 bean 提供了与 `proxyMode` 属性的等效支持; 默认值为无代理 (`ScopedProxyMode.NO`), 但你可以指定 `ScopedProxyMode.TARGET_CLASS`或 `ScopedProxyMode.INTERFACES`  
如果使用 Java 将 XML 参考文档 (参见前面的链接) 中的作用域代理示例移植到我们的 `@Bean`, 它将如下所示
```
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```
##### 自定义 bean 名称
默认情况下, 配置类使用 `@Bean` 方法的名称作为结果 bean 的名称; 但是可以使用 `name` 属性覆盖此功能
```
@Configuration
public class AppConfig {

    @Bean(name = "myFoo")
    public Foo foo() {
        return new Foo();
    }
}
```
##### Bean 别名
正如 [第 7.3.1 节 "命名 bean"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-beanname) 中所讨论的, 有时需要为单个 bean 提供多个名称, 也称为 bean 别名; `@Bean` 注解的 `name` 属性为此接受一个 String 数组
```
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```
##### Bean 描述
有时提供 bean 的更详细的文本描述是有帮助的; 当 bean 暴露(可能通过 JMX ) 用于监视目的时, 这可能特别有用  
要向 `@Bean` 添加描述, 可以使用 `@Description` 注解
```
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Foo foo() {
        return new Foo();
    }
}
```

#### 使用 @Configuration 注解
`@Configuration` 是一个类级别注释, 指示对象是 bean 定义的来源; `@Configuration` 类通过公共 `@Bean` 注释方法声明 bean; 在 `@Configuration` 类上调用 `@Bean` 方法也可用于定义 bean 间依赖关系; 有关介绍请参见 [第 7.12.1 节 "基本概念: `@Bean` 和 `@Configuration`"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-java-basic-concepts)

##### 注入内部 bean 依赖
当 `@Beans` 彼此依赖时, 表达该依赖关系就像让一个 bean 方法调用另一个一样简单
```
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        return new Foo(bar());
    }

    @Bean
    public Bar bar() {
        return new Bar();
    }
}
```
在上面的示例中, `foo` bean 通过构造函数注入接收对 bar 的引用
>这种声明 bean 间依赖关系的方法仅在 `@Configuration` 注解的类中声明 `@Bean` 方法时才有效; 你不能使用普通的 `@Component` 注解的类声明 bean 间依赖关系

##### Lookup 方法注入
如前所述, `Lookup 方法注入` 是一项很少使用的高级功能; 在单例范围的 bean 依赖于原型范围的 bean 的情况下它很有用; 将 Java 用于此类配置提供了实现此模式的自然方法
```
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```
使用 Java 配置支持, 你可以创建 `CommandManager` 的子类, 其中抽象的 `createCommand()` 方法被覆盖, 以便查找新的 (原型) `Command` 对象
```
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with command() overridden
    // to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

##### 有关基于Java 配置如何在内部工作的更多信息
以下示例显示了被调用两次的 `@Bean` 注解方法
```
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```
`clientDao()` 在 `clientService1()` 中调用一次, 在 `clientService2()` 中调用一次; 由于此方法创建 `ClientDaoImpl` 的新实例并返回它, 因此通常期望有 2 个实例 (每个服务一个) 这肯定会有问题: 在 Spring 中, 实例化的 bean 默认具有单例范围; 这就是魔术的来源: 所有 `@Configuration` 注解的类在启动时都使用 `CGLIB` 进行子类化; 在子类中, 子方法在调用父方法并创建新实例之前, 首先检查容器是否有任何缓存 (作用域) bean; 请注意, 从 Spring 3.2 开始, 不再需要将 `CGLIB` 添加到类路径中, 因为 `CGLIB` 类已在 `org.springframework.cglib` 下重新打包并直接包含在 `spring-core` JAR 中
>根据 bean 的范围, 行为可能会有所不同; 我们在这里谈论单例
>由于 CGLIB 在启动时动态添加功能, 因此存在一些限制, 特别是配置类不能是最终的; 但是, 从 4.3 开始, 配置类允许使用任何构造函数, 包括使用 `@Autowired` 或单个非默认构造函数声明进行默认注入
如果你希望避免任何 CGLIB 强加的限制, 请考虑在非 `@Configuration` 类上声明你的 `@Bean` 方法, 例如而是在普通的 `@Component` 类上; 然后 `@Bean` 方法之间的跨方法调用将不会被拦截, 因此你必须完全依赖于构造函数或方法级别的依赖注入

#### 编写基于 Java 的配置
##### 使用 @Import 注解
就像在 Spring XML 文件中使用 `<import/>` 元素来帮助模块化配置一样, `@Immort` 注解允许从另一个配置类加载 `@Bean` 定义
```
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```
现在, 在实例化上下文时, 不需要同时指定 `ConfigA.class` 和 `ConfigB.class`, 只需要显式提供 `ConfigB`:
```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```
这种方法简化了容器实例化, 因为只需要处理一个类, 而不是要求开发人员在构造期间记住可能大量的 `@Configuration` 类
>从 Spring Framework 4.2 开始, `@Immort` 还支持对常规组件类的引用, 类似于 `AnnotationConfigApplicationContext.register` 方法; 如果你想避免组件扫描, 使用一些配置类作为明确定义所有组件的入口点, 这将特别有用

##### 在导入的 @Bean 定义上注入依赖
上面的例子有效, 但很简单; 在大多数实际情况中, bean 将跨配置类相互依赖; 使用 XML 时, 这本身并不是问题, 因为不涉及编译器, 可以简单地声明 `ref="someBean"` 并相信 Spring 会在容器初始化期间解决它; 当然在使用 `@Configuration` 类时, Java 编译器会对配置模型施加约束, 因为对其他 bean 的引用必须是有效的 Java 语法  
幸运的是, 解决这个问题很简单; 正如[我们已经讨论过的](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-java-dependencies), `@Bean` 方法可以有任意数量的参数来描述 bean 的依赖关系; 让我们考虑一个更真实的场景, 其中包含几个 `@Configuration` 类, 每个类都取决于其他类中声明的 bean
```
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```
还有另一种方法可以达到相同的效果; 请记住 `@Configuration` 类最终只是容器中的另一个 bean: 这意味着他们可以像任何其他 bean 一样利用 `@Autowired` 和 `@Value` 注入
>确保以这种方式注入的依赖项只是最简单的类型; `@Configuration` 类在上下文初始化期间很早就被处理, 并且强制以这种方式注入依赖项可能导致意外的早期初始化; 尽可能采用基于参数的注入, 如上例所示
另外, 通过 `@Bean` 特别注意 `BeanPostProcessor` 和 `BeanFactoryPostProcessor` 定义; 这些通常应该声明为静态 `@Bean` 方法, 而不是触发包含它们的配置类实例化; 否则 `@Autowired` 和 `@Value` 将无法在配置类本身上工作, 因为它过早地被创建为 bean 实例
```
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    @Autowired
    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```
现在 `ServiceConfig` 与具体的 `DefaultRepositoryConfig` 松散耦合, 内置的 IDE 工具仍然很有用: 开发人员很容易获得 `RepositoryConfig` 实现的类型层次结构; 通过这种方式, 引导 `@Configuration`类及其依赖项与引导基于接口的代码的常规过程没有什么不同

##### 有条件地包括 `@Configuration` 类或 `@Bean` 方法
基于某些任意系统状态, 有条件地启用或禁用完整的 `@Configuration` 类, 甚至单独的 `@Bean` 方法通常很有用; 一个常见的例子是只有在 Spring 环境中启用了特定的配置文件时才使用 `@Profile` 注解激活的 bean (有关详细信息, 请参见 [第 7.13.1节 "Bean 定义配置文件"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-definition-profiles))  
`@Profile` 注解实际上是使用一个名为 `@Conditional` 的更灵活的注解实现的; `@Conditional` 注解表示在注册 `@Bean` 之前应该参考的特定 `org.springframework.context.annotation.Condition` 实现  
`Condition` 接口的实现只提供一个返回 true 或 false 的 `matches(...)` 方法; 例如, 以下是用于 `@Profile` 的实际 `Condition` 实现
```
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    if (context.getEnvironment() != null) {
        // Read the @Profile annotation attributes
        MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
        if (attrs != null) {
            for (Object value : attrs.get("value")) {
                if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                    return true;
                }
            }
            return false;
        }
    }
    return true;
}
```
有关更多详细信息, 请参阅 [@Conditional javadocs](https://docs.spring.io/spring-framework/docs/4.3.24.RELEASE/javadoc-api/org/springframework/context/annotation/Conditional.html)
##### Java 和 XML 的配置
Spring 的 `@Configuration` 类支持并非旨在成为 Spring XML 的 100% 完全替代品; 诸如 Spring XML 命名空间之类的一些工具仍然是配置容器的理想方式; 在 XML 方便或必要的情况下, 你可以选择: 使用例如 `ClassPathXmlApplicationContext` 以 "以XML为中心" 的方式实例化容器, 或者使用 `AnnotationConfigApplicationContext` 和 `@ImportResource` 注解以 "以 Java 为中心" 的方式实例化容器, 根据需要导入 XML

##### 以 XML 为中心的 `@Configuration` 类的使用
最好从 XML 引导 Spring 容器, 并以 ad-hoc 方式包含 `@Configuration` 类; 例如, 在使用 Spring XML 的大型现有代码库中, 根据需要创建 `@Configuration` 类并将其包含在现有 XML 文件中会更容易; 下面你将找到在这种 "以 XML 为中心" 的情况下使用 `@Configuration` 注解类的选项  
请记住, `@Configuration` 类最终只是容器中的 bean 定义; 在此示例中, 我们创建一个名为 `AppConfig` 的 `@Configuration` 类, 并将其作为 `<bean/>` 定义包含在 `system-test-config.xml` 中; 由于 `<context:annotation-config/>` 已打开, 容器将识别 `@Configuration` 注解并正确处理 `AppConfig` 中声明的 `@Bean` 方法
```
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }
}
```
system-test-config.xml
```
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```
jdbc.properties
```
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```
```
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```
>在上面的 `system-test-config.xml` 中, `AppConfig` 的 `<bean/>` 不声明 `id` 元素; 虽然这样做是可以接受的, 但是没有其他 bean 可以引用它, 并且不太可能通过名称从容器中显式获取它; 与 `DataSource` bean 类似, 它只是按类型自动装配, 因此不严格要求显式 bean ID

因为 `@Configuration` 是使用 `@Component` 进行元注释的, 所以 `@Configuration-annotated` 类自动成为组件扫描的候选者; 使用与上面相同的方案, 我们可以重新定义 `system-test-config.xml` 以利用组件扫描; 请注意, 在这种情况下, 我们不需要显式声明 `<context：annotation-config/>`, 因为 `<context:component-scan/>` 启用相同的功能  

```
# system-test-config.xml
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```
##### `@Configuration` 以类为中心使用带 `@ImportResource` 的 XML
在 `@Configuration` 类是配置容器的主要机制的应用程序中, 仍然可能需要使用至少一些 XML; 在这些场景中, 只需使用 `@ImportResource` 并仅根据需要定义尽可能多的 XML; 这样做可以实现 "以 Java 为中心" 的方法来配置容器并将 XML 保持在最低限度
```
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}
```
```
# properties-config.xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```
```
jdbc.properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```
```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

>**参考:**  
[Java-based container configuration](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-java)
