### 基于注解的容器配置
>配置 Spring 使用注解是否比 XML 更好
基于注释的配置的引入引出了这种方法是否比 XML 更 "好" 的问题; 简短的答案取决于场景; 长答案是每种方法都有其优点和缺点,通常由开发人员决定哪种策略更适合他们; 由于它们的定义方式, 注释在其声明中提供了大量上下文, 从而导致更短更简洁的配置; 但是, XML擅长在不触及源代码或重新编译它们的情况下连接组件; 一些开发人员更喜欢靠近源进行配置, 而另一些开发人员则认为注释类不再是 POJO, 而且配置变得分散且难以控制
无论选择如何, Spring 都可以兼顾两种风格, 甚至可以将它们混合在一起; 值得指出的是, 通过其 [JavaConfig](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-java) 选项, Spring 允许以非侵入方式使用注解, 而无需触及目标组件源代码, 并且在工具方面, [Spring Tool Suite](https://spring.io/tools/sts) 支持所有配置样式

基于注释的配置提供了 XML 设置的替代方案, 该配置依赖于字节码元数据来连接组件而不是角括号声明; 开发人员不是使用 XML 来描述 bean 连接, 而是通过在相关的类, 方法或字段声明上使用注释将配置移动到组件类本身; 正如在 ["示例: RequiredAnnotationBeanPostProcessor" 一节](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-extension-bpp-examples-rabpp) 中所提到的, 将 `BeanPostProcessor` 与注释结合使用是扩展 Spring IoC 容器的常用方法; 例如, Spring 2.0 引入了使用 `@Required` 注解强制执行所需属性的可能性; Spring 2.5 使得有可能采用相同的通用方法来驱动 Spring 的依赖注入; 从本质上讲, `@Autowired` 注释提供的功能与 [第 7.4.5 节 "自动装配协作者" 中](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-autowire) 描述的功能相同, 但具有更细粒度的控制和更广泛的适用性; Spring 2.5 还增加了对 `JSR-250` 注解的支持, 例如 `@PostConstruct` 和 `@PreDestroy`; Spring 3.0 增加了对 javax.inject 包中包含的 `JSR-330` (Java 的依赖注入) 注解的支持, 例如 `@Inject` 和 `@Named`; 有关这些注释的详细信息, 请参阅 [相关章节](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-standard-annotations)  
>注释注入在 XML 注入之前执行, 因此后一种配置将覆盖通过两种方法连接的属性的前者

与之前一样, 你可以将它们注册为单独的 bean 定义, 但也可以通过在基于 XML 的 Spring 配置中包含以下标记来隐式注册它们 (请注意包含上下文命名空间)
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```
(隐式注册的 post-processors 包括 `AutowiredAnnotationBeanPostProcessor`, `CommonAnnotationBeanPostProcessor`, `PersistenceAnnotationBeanPostProcessor`, 以及前面提到的 `RequiredAnnotationBeanPostProcessor`)
>`<context:annotation-config/>` 仅在定义它的同一应用程序上下文中查找 bean 上的注解; 这意味着, 如果将 `<context:annotation-config/>` 放在 `DispatcherServlet` 的 `WebApplicationContext` 中, 它只检查控制器层中的 `@Autowired` bean, 而不检查你的服务层; 有关更多信息, 请参见 [第 22.2 节 "DispatcherServlet"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/mvc.html#mvc-servlet)

#### @Required
`@Required` 注解引用于 bean 属性的 setter 方法, 如以下示例所示
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```
此注释仅表示受影响的 bean 属性必须在配置时填充, 通过 bean 定义中的显式属性值或通过自动装配填充; 如果尚未填充受影响的 bean 属性, 容器将引发异常; 这允许急切和明确的失败, 以后避免 `NullPointerException` 等; 仍然建议你将断言放入 bean 类本身, 例如, 放入 `init` 方法; 即使你在容器外部使用类, 这样做也会强制执行那些必需的引用和值

#### @Autowired
>在下面的示例中, 可以使用 `JSR-330` 的 `@Inject` 注解代替 Spring 的 `@Autowired` 注解; 有关详细信息, 请参见 [此处](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-standard-annotations)

你可以在构造器上应用 `@Autowired`
```
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```
>从 Spring Framework 4.3 开始, 如果目标 bean 只定义了一个构造函数, 则不再需要在这样的构造函数上使用 `@Autowired` 注解; 但是, 如果有几个构造器可用, 则必须注释至少一个构造器以指导容器使用哪一个

正如所料, 你还可以将 `@Autowired` 注解应用于 "传统" 的 setter 方法
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```
你还可以将注释应用于具有任意名称或多个参数的方法
```
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```
你也可以将 `@Autowired` 应用于字段, 甚至将其与构造函数混合
```
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```
>确保你的目标组件 (例如 `MovieCatalogm, CustomerPreferenceDao`) 始终按照你用于 `@Autowired` 注解注入点的类型进行声明; 否则, 由于在运行时未找到类型匹配, 注入可能会失败
对于通过类路径扫描找到的 XML 定义的 bean 或组件类, 容器通常预先知道具体类型; 但是, 对于 `@Bean` 工厂方法, 你需要确保声明的返回类型具有足够的表现力; 对于实现多个接口的组件或可能由其实现类型引用的组件, 请考虑在工厂方法上声明最具体的返回类型 (至少与引用 bean 的注入点所要求的具体相同)

通过将注解添加到需要该类型数组的字段或方法, 可以从 `ApplicationContext` 提供特定类型的所有 bean
```
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```
这同样适用于类型集合
```
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```
>如果希望数组或列表中的项按特定顺序排序, 则目标 bean 可以实现 `org.springframework.core.Ordered` 接口或使用 `@Order` 或标准 `@Priority` 注解; 否则, 它们的顺序将遵循容器中相应目标 bean 定义的注册顺序
`@Order` 注解可以在目标类级别声明, 也可以在 `@Bean` 方法上声明, 可能是每个 bean 定义非常独立 (如果具有相同 bean 类的多个定义); `@Order` 值可能影响注入点的优先级, 但请注意它们不会影响单例启动顺序, 这是由依赖关系和 `@DependsOn` 声明确定的正交关注点
请注意, 标准的 `javax.annotation.Priority` 注释在 `@Bean` 级别不可用, 因为它无法在方法上声明; 它的语义可以通过 `@Order` 值与每个类型的单个 bean 上的 `@Primary` 一起建模

只要预期的键类型是 String, 即使是类型化的 Map 也可以自动装配; Map 值将包含所需类型的所有 bean, 并且键将包含相应的 bean 名称
```
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```
默认情况下, 只要没有候选 bean 可用, 自动装配就会失败; 默认行为是将带注解的方法, 构造函数和字段视为指示所需的依赖项; 可以更改此行为, 如下所示
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```
>每个类只能标记一个带注解的构造函数, 但可以注解多个非必需的构造函数; 在这种情况下, 每个都被认为是候选者之一, Spring使用最贪婪的构造函数, 其依赖性可以得到满足, 即具有最多参数的构造函数
建议使用 `@Autowired` 的 `required` 属性而不是 `@Required` 注释; `required` 属性表示该属性不是自动装配所必需的, 如果无法自动装配则会忽略该属性; 另一方面, `@Required` 更强大, 因为它强制执行由容器支持的任何方式设置的属性; 如果未注入任何值, 则会引发相应的异常

或者，你可以通过 Java 8 的 `java.util.Optional` 表达特定依赖项的非必需特性
```
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```
你还可以将 `@Autowired` 用于众所周知的可解析依赖项的接口: `BeanFactory, ApplicationContext, Environment, ResourceLoader, ApplicationEventPublisher, MessageSource` 这些接口及其扩展接口(如 `ConfigurableApplicationContext` 或 `ResourcePatternResolver`) 将自动解析, 无需特殊设置
```
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```
>`@Autowired`, `@Inject`, `@Resource` 和 `@Value` 注解由 Spring BeanPostProcessor 实现处理, s这反过来意味着你不能在自己的 BeanPostProcessor 或 BeanFactoryPostProcessor 类型 (如果有) 中应用这些注释; 必须通过 XML 或使用 Spring `@Bean` 方法显式地 "连接" 这些类型

#### 使用 @Primary 的细粒度的基于注解的自动装配

>**参考:**  
[Annotation-based container configuration](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-annotation-config)
