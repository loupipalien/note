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
由于按类型自动装配可能会导致多个候选, 因此通常需要对选择过程进行更多控制; 实现这一目标的一种方法是使用 Spring的 `@Primary` 注解; `@Primary` 指示当多个 bean 可以自动装配到单值依赖项时, 应该优先选择特定的 bean; 如果候选者中只存在一个 "主要的" bean, 则它将是自动装配的值  
假设我们有以下配置将 `firstMovieCatalog` 定义为主要的 `MovieCatalog`
```
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```
通过这样的配置, 以下 `MovieRecommender` 将与 `firstMovieCatalog` 一起自动装配
```
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```
相应的 XML 配置 bean 定义如下所示
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

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```
`@Qualifier` 注解也可以在各个构造函数参数或方法参数上指定
```
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main")MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```
相应的 XML 配置 bean 定义如下所示; 具有限定符值 "main" 的 bean 与使用相同值限定的构造函数参数连接
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

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```
对于回退匹配, bean 名称被视为默认限定符值; 因此, 你可以使用 `id` 为 "main" 而不是嵌套的限定符元素来定义 bean, 从而得到相同的匹配结果; 但是, 虽然你可以使用此约定来按名称引用特定 bean, 但 `@Autowired` 基本上是关于具有可选语义限定符的类型驱动注入; 这意味着即使使用 bean 名称回退, 限定符值在类型匹配集中也总是具有缩小的语义; 它们在语义上不表示对唯一 bean `id` 的引用; 好的限定符值是 "main" 或 "EMEA" 或 "persistent", 表示独立于 bean `id` 的特定组件的特征, 在匿名 bean 定义的情况下可以自动生成, 如上例中的那个  
限定符也适用于类型化集合, 如上所述, 例如: `Set<MovieCatalog>`; 在这种情况下, 根据声明的限定符的所有匹配 bean 都作为集合注入; 这意味着限定符不必是唯一的, 它们只是简单地构成过滤标准; 例如, 你可以使用相同的限定符值 "action" 定义多个 `MovieCatalog` bean, 所有这些 bean 都将注入到使用 `@Qualifier("action")` 注解的 `Set<MovieCatalog>` 中
>在类型匹配候选项中, 根据目标 bean 名称选择限定符值, 甚至不需要在注入点处使用 `@Qualifier` 注解; 如果没有其他分辨率指示符 (例如限定符或主要标记), 对于非唯一依赖性情况, Spring 将使注入点名称 (即字段名称或参数名称) 与目标 bean 名称匹配, 如果有候选人则会选择相同的  
也就是说, 如果你打算按名称表达注释驱动的注入, 请不要主要使用 `@Autowired`, 即使能够在类型匹配候选项中通过 bean 名称进行选择; 相反, 使用 `JSR-250` 的 `@Resource` 注解, 该注解在语义上定义为通过其唯一名称标识特定目标组件, 声明的类型与匹配过程无关; `@Autowired` 具有相当不同的语义: 在按类型选择候选 bean 之后, 将仅在那些类型选择的候选者中考虑指定的字符串限定符值, 例如, 将 "account" 限定符与标记有相同限定符标签的 bean 匹配
对于本身定义为集合/映射或数组类型的 bean, `@Resource` 是一个很好的解决方案, 通过唯一名称引用特定的集合或数组 bean; 也就是说, 从 4.3 开始, 只要元素类型信息保存在 `@Bean` 返回类型签名或集合继承层次结构中, 集合/映射和数组类型也可以通过 Spring的 `@Autowired` 类型匹配算法进行匹配; 在这种情况下, 限定符值可用于在相同类型的集合中进行选择, 如上一段所述
从 4.3 开始, `@Autowired` 还考虑了自我引用注入, 即引用回到当前注入的 bean; 请注意, 自我注射是一种后备; 对其他组件的常规依赖性始终具有优先权; 从这个意义上讲, 自我引用并不参与常规的候选人选择, 因此尤其不是主要的; 相反, 它们总是最低优先级; 在实践中, 仅使用自引用作为最后的手段, 例如通过 bean 的事务代理调用同一实例上的其他方法: 在这种情况下, 考虑将受影响的方法分解为单独的委托 bean; 或者使用 `@Resource`, 它可以通过其唯一名称获取代理回到当前 bean
`@Autowired` 适用于字段, 构造函数和多参数方法, 允许在参数级别缩小限定符注解; 相比之下, `@Resource` 仅支持具有单个参数的字段和 bean 属性 setter 方法; 因此, 如果你的注射目标是构造函数或多参数方法, 请坚持使用限定符

你可以创建自己的自定义限定符注解, 只需定义注解并在定义中提供 `@Qualifier` 注解
```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```
然后你可以在自动装配的字段和参数上提供自定义限定符
```
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...
}
```
接下来, 提供候选 bean 定义的信息; 你可以将 `<qualifier/>` 标记添加为 `<bean/>` 标记的子元素, 然后指定与自定义限定符注解匹配的类型和值; 类型与注解的完全限定类名匹配; 或者, 为方便起见, 如果不存在冲突名称的风险, 你可以使用短类名称; 以下示例演示了这两种方法
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

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```
在 [第 7.10 节 "类路径扫描和托管组件"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-classpath-scanning) 中, 你将看到基于注解的替代方法, 用于在 XML 中提供限定符元数据; 具体来说, 请参见 [第 7.10.8 节 "使用注释提供限定符元数据"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-scanning-qualifiers)  

在某些情况下, 使用没有值的注解可能就足够了; 当注解用于更通用的目的并且可以跨多种不同类型的依赖项应用时, 这可能很有用; 例如, 你可以提供在没有 Internet 连接时将搜索的脱机目录; 首先定义简单注释
```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {

}
```
然后将注解添加到要自动装配的字段或属性
```
public class MovieRecommender {

    @Autowired
    @Offline
    private MovieCatalog offlineCatalog;

    // ...
}
```
现在 bean 定义只需要一个限定符类型
```
<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/>
    <!-- inject any dependencies required by this bean -->
</bean>
```
你还可以定义除简单值属性之外或代替简单值属性接受命名属性的自定义限定符注解; 如果随后在要自动装配的字段或参数上指定了多个属性值, 则 bean 定义必须匹配所有此类属性值才能被视为自动装配候选; 例如, 请考虑以下注释定义
```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();
}
```
在这种情况下 `Format` 是一个枚举
```
public enum Format {
    VHS, DVD, BLURAY
}
```
要自动装配的字段使用自定义限定符进行注解, 并包含两个属性的值: `genre` 和 `format`
```
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...
}
```
最后, bean 定义应包含匹配的限定符值; 此示例还演示了可以使用 bean 元属性而不是 `<qualifier/>` 子元素; 如果可用, `<qualifier/>` 及其属性优先, 但如果不存在此类限定符, 则自动装配机制将回退到 `<meta/>` 标记内提供的值, 如以下示例中的最后两个 bean 定义
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

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

</beans>
```

#### 使用泛型作为自动装配限定符
除了 `@Qualifier` 注解之外, 还可以使用 Java 泛型类型作为隐式形式的限定; 例如, 假设你具有以下配置
```
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```
假设上面的 bean 实现了一个通用接口, 即 `Store<String>` 和 `Store <Integer>`, 你可以 `@Autowire` Store 接口, 泛型将被用作限定符
```
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```
通用限定符也适用于自动装配列表, MAP 和数组
```
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

#### CustomAutowireConfigurer
`CustomAutowireConfigurer` 是一个 `BeanFactoryPostProcessor`, 它允许你注册自己的自定义限定符注释类型， 即使它们没有使用 Spring 的 `@Qualifier` 进行注解
```
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```
`AutowireCandidateResolver` 通过 autowire 确定候选者
- 每个 bean 定义的 `autowire-candidate` 值
- `<beans/>` 元素上任何 `default-autowire-candidates` 配置值都可用
- `@Qualifier` 注解的存在以及在 `CustomAutowireConfigurer` 中注册的任何自定义注解

当多个 bean 有资格作为 `autowire` 候选者时, "primary" 的确定如下: 如果候选者中只有一个 bean 定义的主要属性设置为 true, 则将选择它

#### @Resource
Spring 还支持在字段或 bean 属性 `setter` 方法上使用 `JSR-250` 的 `@Resource` 注解进行注入; 这是 Java EE 5 和 6 中的常见模式, 例如在 JSF 1.2 托管 bean 或 JAX-WS 2.0 端点中; Spring 也支持 Spring 管理对象的这种模式  
`@Resource` 采用 name 属性, 默认情况下, Spring 将该值解释为要注入的 bean 名称; 换句话说, 它遵循按名称语义, 如本例所示
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder")
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```
如果未明确指定名称, 则默认名称是从字段名称或 `setter` 方法派生的; 如果是字段, 则采用字段名称; 在 `setter` 方法的情况下, 它采用 bean 属性名称; 所以下面的例子将把名为  "movieFinder" 的 bean 注入其 `setter` 方法
```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```
>随注解提供的名称由 `ApplicationContext` 解析为 bean 名称, `CommonAnnotationBeanPostProcessor` 知道该名称; 如果你显式配置 Spring的 `SimpleJndiBeanFactory`, 则可以通过 JNDI 解析名称; 但是, 建议你依赖于默认行为， 只需使用 Spring 的 JNDI 查找功能来保持间接级别

在没有指定显式名称且类似于 `@Autowired` 的 `@Resource` 用法的独占情况下, `@Resource` 找到主要类型匹配而不是特定的命名 bean, 并解析众所周知的可解析依赖项: `BeanFactorym, ApplicationContext, ResourceLoader, ApplicationEventPublisher, , MessageSource` 接口  
因此在以下示例中, `customerPreferenceDao` 字段首先查找名为 `customerPreferenceDao` 的 bean, 然后返回 `CustomerPreferenceDao` 类型的主类型匹配; 基于已知的可解析依赖类型 `ApplicationContext` 注入 "context" 字段
```
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

#### @PostConstruct 和 @PreDestroy
`CommonAnnotationBeanPostProcessor` 不仅识别 `@Resource` 注解, 还识别 `JSR-250` 生命周期注解; 在 Spring 2.5 中引入, 对这些注解的支持提供了初始化回调和销毁回调中描述的另一种替代方法; 如果 `CommonAnnotationBeanPostProcessor` 在 Spring 的 `ApplicationContext` 中注册, 则在生命周期的同一点调用承载这些注释之一的方法, 作为相应的 Spring 生命周期接口方法或显式声明的回调方法; 在下面的示例中, 缓存将在初始化时预先填充, 并在销毁时清除
```
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```
>有关组合各种生命周期机制的效果的详细信息, 请参阅 [组合生命周期机制](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-lifecycle-combined-effects) 一节

>**参考:**  
[Annotation-based container configuration](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-annotation-config)
