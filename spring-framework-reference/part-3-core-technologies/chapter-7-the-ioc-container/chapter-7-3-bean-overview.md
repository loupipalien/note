#### Bean 概览
一个 Spring IoC 容器管理了一个或多个 beans; 这些 beans 是根据你提供给容器的元数据创建的; 例如以 XML `<bean/>` 定义的形式  
在容器中, bean 定义由 `BeanDefinition` 对象表示, 它包含了以下元数据 (即其他信息):
- 一个包级的限定类名: 通常是定义 bean 的实际实现类
- Bean 的行为配置元素, 即表示了 bean 在容器中的行为方式 (scope, lifecycle, callbacks 等等)
- 此 bean 工作需要的其他 bean 的引用; 这些引用被称作协作者或依赖
- 其他在新创建的对象中的设置, 例如在管理连接池的 Bean 中使用的连接数, 或池的大小限制

此元数据转换为构成每个 bean 定义的一组属性

| 属性 | 解释章节 |
| :--- | :--- |
| class | [7.3.2 小节, beans 的实例化](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-class) |
| name | [7.3.1 小节, beans 的命名](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-beanname)  |
| scope | [7.5 小节, Bean 范围](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes) |
| constuctor arguments | [7.4.1 小节, 依赖注入](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-collaborators) |
| properties | [7.4.1 小节, 依赖注入](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-collaborators) |
| autowiring mode | [7.4.5 小节, 自动装配协作者](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-autowire) |
| lazy-initialization mode | [7.4.4 小节, 懒加载 beans](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-lazy-init) |
| initialization method | [实例化回调小节](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-lifecycle-initializingbean) |
| destruction method | [销毁回调小节](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-lifecycle-disposablebean)  |

另外, bean 定义包含了如何创建一个指定 bean 的信息, `ApplicationContext` 实现也允许注册用户在容器外部创建的已存在的对象; 这可以通过 `getBeanFactory()` 方法访问 `ApplicationContext` 的 `BeanFactory` 来完成, 该方法返回 `BeanFactory` 的实现 `DefaultListableBeanFactory`; `DefaultListableBeanFactory `通过方法 `registerSingleton(..)` 和 `registerBeanDefinition(..)` 支持此注册; 但通常应用程序仅通过元数据 bean定义来定义 bean
>需要尽早注册 Bean 元数据和手动提供的单例实例, 以便容器在自动装配和其他内省步骤期间正确推理它们, 虽然在某种程度上支持覆盖现有元数据和现有单例实例, 但是在运行时注册新 bean (与对工厂的实时访问同时) 并未得到官方支持, 并且可能导致 bean 容器中的并发访问异常或者不一致状态

##### 命名 beans
每个 bean 有一个或多个标识符, 这些标识符在 bean 托管的容器中必须是唯一的; 一个 bean 通常只有一个标识符, 但如果要求超过一个, 额外的可以认为是别名  
在基于 XML 的配置元数据中, 你可以使用 id 或者 name 属性来指定 bean 的标识符; `id` 属性仅允许你指定一个; 名称约定是字母表示的 ('myBean', 'fooService', 等等), 但也可以包含特殊字符; 如果你想为 bean 指定其他的别名, 你也可以在 `name` 属性中使用逗号, 分号, 或空格分隔指定; 由于历史原因, 在 Spring 3.1 之前的版本中, `id` 属性被定义为 `xsd：ID` 类型用来约束可能的字符; 从 3.1 开始, 它被定义为 `xsd：string`类型;  请注意 bean `id` 的唯一性仍由容器强制执行, 不再由 XML 解析器执行  
你不需要为一个 bean 提供名称或 id; 如果没有显式提供名称或 id, 容器会为该 bean 生成唯一的名称; 但是, 如果要通过名称引用该 bean, 会通过使用 ref 元素或 [Service Locator](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-servicelocator) 样式, 所以你必须提供一个名称; 不提供名称的动机与使用 [内部 bean](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-inner-beans) 和 [自动装配协作者](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-autowire) 有关
```
Bean 命名约定
命名 bean 时使用标准 Java 对实例字段名的约定; 那就是 bean 名以小写字母开头, 并且是驼峰式; 例如这样的名称 (没有单引号) 'accountManager', 'accountService', 'userDao', 'loginController' 等等
一致性的 bean 命名使得你的配置更易读和理解, 并且如果你使用 Spring AOP, 当你根据名称应用通知到一系列的 beans 时这会很有帮助
```
>使用类路径中的组件扫描, Spring 按照上述规则为未命名的组件生成 bean 名称: 实际上, 采用简单的类名并将其初始字符转换为小写; 但是在 (不常见的) 特殊情况下, 当有多个字符且第一个和第二个字符都是大写字母时, 原始外壳将被保留;  这些规则与 java.beans.Introspector.decapitalize (Spring 在此处使用) 中定义的规则相同

###### bean 定义之外定义 bean 的别名
在 bean 定义中你可以为 bean 提供多个名称, 方法是使用 `id` 属性指定的最多一个名称和 `name` 属性中的任意数量的其他名称; 这些名称可以是同一个 bean 的等效别名, 并且在某些情况下很有用, 例如允许应用程序中的每个组件通过使用特定于该组件本身的 bean 名称来引用公共依赖  
指定实际定义 bean 的所有别名并不常见; 但是有时需要为其他地方定义的 bean 引入别名; 在大型系统中这种情况较为常见, 其配置分隔在每个子系统中, 每个子系统具有其自己的一组对象定义; 在基于 XML 的配置元数据中, 你可以使用 `<alias/>` 元素来完成
```
<alias name="fromName" alias="toName"/>
```
在这种情况下, 使用此别名定义之后, 同名容器中名为 `fromName` 的 bean 也可以称为 `toName`  
例如, 子系统 A 的配置元数据可以通过 `subsystemA-dataSource` 名称引用 DataSource; 子系统 B 的配置元数据可以通过 `subsystemB-dataSource` 名称引用 DataSource; 在编写使用这两个子系统的主应用程序时, 主应用程序通过名称 `myApp-dataSource` 引用 DataSource; 要让所有三个名称指向你添加到 MyApp 配置元数据的同一对象, 请使用以下别名定义
```
<alias name="subsystemA-dataSource" alias="subsystemB-dataSource"/>
<alias name="subsystemA-dataSource" alias="myApp-dataSource" />
```
现在, 每个组件和主应用程序都可以通过一个唯一的名称引用 DataSource, 并保证不与任何其他定义冲突 (有效地创建命名空间), 但它们引用相同的 bean
```
Java-configuration
如果你使用 Java-configuration, `@Bean` 注解可以用于提供别名, 详细见 [7.12.3 小节, 使用 `@Bean` 注解](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-java-bean-annotation)
```

##### 实例化 beans
bean 定义本质上是用于创建一个或多个对象的配方; 当一个命名 bean 被查找时容器查看配方, 并且使用 bean 定义封装的配置元数据来创建 (或获取) 一个真正的对象  
如果你使用基于 XML 的配置元数据, 在 `<bean/>` 元素的 `class` 属性中指定要实例化的对象的类型 (或类); `class` 属性, 在 `BeanDefinition` 的内部是一个 `Class` 类型的属性, 此属性是必须的; (例外情况, 见 [使用一个实例工厂方法初始化小节](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-class-instance-factory-method) 和 [7.7 小节, Bean 定义继承](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-child-bean-definitions)) 你以以下两种方式使用 `Class` 属性
- 通常, 容器本身通过反向调用其构造函数直接创建 bean 的情况下指定要构造的 bean 类, 一定程度上等同于使用 `new`运算符的 Java 代码
- 指定包含将被调用以创建对象的静态工厂方法的实际类, 很少的情况下, 容器会在类上调用静态工厂方法来创建 bean; 从静态工厂方法的调用返回的对象类型完全可以是同一个类或另一个类
```
内部类名称: 如果你想为一个静态内部类配置一个 bean 定义, 你需要使用内部类的二进制名称
例如, 如果在 `com.example` 包中你有一个叫做 `Foo` 的类, 并且这个 `Foo` 类有一个叫做 `Bar` 的静态内部类, 则在 bean 定义中的 `class` 属性的值将会是 `com.example.Foo$Bar`; 注意使用 `$` 字符分隔外部类名称和内部类名称
```

###### 使用构造器实例化
当你通过构造器方法创建一个 bean 时, 所有普通类都可以使用并且与 Spring 兼容; 这样类在被开发时不需要实现任何指定的接口或者编写一个指定的方法; 简单的指定 bean 就足够了; 然而, 根据你为该特定 bean 使用的 IoC 类型, 你可能需要一个默认的 (空的) 构造器  
Spring IoC 容器事实上可以管理逆向管理的任何类; 它不仅限于管理真正的 JavaBeans; 大多数 Spring 用户更喜欢具有默认 (无参数) 构造函数的 JavaBeans, 并为容器中的属性正确的调用 setter和 getter; 你还可以在容器中拥有更多别样的非 bean 样式类; 例如, 如果你需要使用不符合 JavaBean 规范的遗留连接池, Spring 也可以对其进行管理  
基于 XML 的配置元数据, 你可以如下方式指定你的 bean
```
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```
有关为构造函数提供参数的机制 (如果需要) 以及在构造对象后设置对象实例属性的详细信息, 见 [注入依赖](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-collaborators)

###### 使用静态工厂方法初始化
定义使用静态工厂方法创建的 bean 时, 你可以使用 `class`属性指定包含静态工厂方法的类和 `factory-method` 的属性指定工厂方法的名称; 你应该能够调用此方法 (使用后面描述的可选参数) 并返回一个活动对象, 之后将其视为通过构造函数创建的对象; 这种 bean 定义的一个用途是调用在遗留代码中的静态工厂方法  
以下 bea n定义指定通过调用 `factory-method` 创建 bean; 该定义未指定返回对象的类型 (类), 仅指定包含工厂方法的类; 在此示例中, `createInstance()` 方法一定是静态方法
```
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```
```
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```
有关在从工厂返回对象后向工厂方法提供 (可选) 参数并设置对象实例属性的机制的详细信息, 详见 [依赖和配置](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-properties-detailed)

###### 使用实例工厂方法实例化
与通过静态工厂方法实例化类似, 使用实例工厂方法进行实例化会从容器调用现有 bean 的非静态方法来创建新 bean; 要使用此机制, 请将 `class` 属性保留为空, 并在 `factory-bean` 属性中指定当前 (或父/祖先) 容器中 bean 的名称, 该容器包含要调用的来创建对象的实例方法; 使用 `factory-method` 属性设置工厂方法的名称
```
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```
```
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```
一个工厂类也可以包含多个工厂方法, 如下所示
```
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```
```
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```
这种方法表明工厂 bean 本身可以通过依赖注入 (DI) 进行管理和配置; 详见 [依赖和配置](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-properties-detailed)

>**参考:**  
[Bean overview](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-definition)
