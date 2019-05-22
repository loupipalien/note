### 依赖
通常企业级的应用程序不会由单个对象组成 (或 Spring 用法中的 bean); 即使最简单的应用也是有一些对象共同协作展现给终端用户一个整体应用; 下一节将介绍如何定义多个独立的 bean 定义, 到对象协作达成目标的完全实现的应用程序

#### 依赖注入
依赖注入 (DI) 是对象定义它们依赖的过程, 即与之工作的其他对象; 在它被构造后或者从工厂方法返回后, 只能通过构造器方法, 工厂方法参数, 或者对象实例的属性设置; 容器创建这个 bean 时就注入这些依赖; 这个过程本质上是反向的, 因此也叫做控制反转 (IoC), bean 本身通过使用类的直接构造器或 Service Locator 模式来控制其依赖的实例化或定位其依赖  
使用 DI 机制的代码更清晰, 并且当对象提供其依赖项时, 解耦更有效; 对象不会查找它的依赖, 并且也不知道依赖的位置和类; 这样你的类变得更易测试, 尤其是当依赖是接口或者抽象基类时, 这允许在单元测试中使用存根或模拟实现  
DI 主要存在两种变体, [基于构造器的依赖注入](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-constructor-injection) 和 [基于 Setter 的依赖注入](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-setter-injection)

##### 基于构造器的依赖注入
基于构造器的 DI 由容器调用具有多个参数的构造函数来完成, 每个参数表示一个依赖项; 与调用具有特定参数的静态工厂方法来构造 bean 几乎是等效的, 本讨论对待构造器和静态工厂方法的参数是类似的; 以下示例显示了一个只能通过构造器注入进行依赖注入的类; 请注意, 此类没有什么特别之处, 它是一个不依赖于容器特定的接口,基类,注释的 POJO
```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```
##### 构造器参数解析
使用参数的类型进行构造器参数解析匹配; 如果 bean 定义的构造函数参数中不存在潜在的歧义, 那么在 bean 定义中定义的构造器参数的顺序是在实例化 bean 时将这些参数提供给适当的构造函数的顺序; 考虑以下类
```
package x.y;

public class Foo {

    public Foo(Bar bar, Baz baz) {
        // ...
    }
}
```
假设 Bar 和 Baz 类不存在继承关系, 则不存在潜在的歧义; 因此, 以下配置正常工作, 你无需在 `<constructor-arg/>` 元素中显式指定构造函数参数索引或类型
```
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>

    <bean id="bar" class="x.y.Bar"/>

    <bean id="baz" class="x.y.Baz"/>
</beans>
```
当引用另一个 bean 的类型是已知的, 并且可以发生匹配 (与前面的示例一样); 当使用简单类型时, 例如 `<value>true</ value>`, Spring 无法确定值的类型, 并且在没有类型的帮助下无法匹配; 考虑以下类:
```
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```
在以上的场景中, 如果你使用 type 属性显式指定构造器参数的类型, 则容器可以使用类型与简单类型匹配; 例如:
```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```
使用 `index` 属性可以明确指定构造器参数的索引; 例如:
```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```
除了解析多个简单值的歧义之外, 指定索引还可以解决构造函数具有相同类型的两个参数的歧义; 请注意索引是基于 0 的  
你也可以使用构造器参数名进行值消歧:
```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```
请注意为了使这可开箱即用, 你的代码必须在启用调试标志 ( -parameters) 的情况下编译, 以便 Spring 可以从构造函数中查找参数名称; 如果无法使用 debug 标志 (或不想) 编译代码, 则可以使用 `@ConstructorProperties` JDK 注解显式命名构造函数参数; 示例类如下所示:
```
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

##### 基于 Setter 的依赖注解
基于 Setter 的 DI 是在调用无参数构造器或无参数静态工厂方法来实例化 bean 之后, 通过容器调用 bean 上的 setter 方法来完成的  
以下示例显示了一个只能使用纯 setter 注入进行依赖注入的类; 这个类是传统的 Java 类; 它是一个不依赖于容器特定的接口, 基类, 注释的 POJO
```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```
`ApplicationContext` 支持它管理的 bean 基于构造器和基于 Setter 的 DI; 它还支持在通过构造器方法注入了一些依赖项之后, 基于 setter 的 DI; 你可以以 BeanDefinition 的形式配置依赖项, 并将其与 `PropertyEditor` 实例结合使用, 以将属性从一种格式转换为另一种格式; 但大多数 Spring 用户不直接使用这些类 (即以编程方式), 而是使用 XML 的 `bean` 定义, 带注释的组件 (即使用 `@Component`, `@Controller` 等注解的类) 或基于 Java的 `@Configuration` 类中的 `@Bean` 方法; 而后这些源在内部转换为 BeanDefinition 的实例, 并用于加载整个 Spring IoC 容器实例
```
基于构造器或基于 setter 的 DI?
由于你可以混合基于构造器和基于 setter 的 DI, 因此将构造函数用于强制依赖和 setter 方法或配置方法用于可选依赖是一个很好的经验法则; 请注意, 在 setter 方法上使用 [@Required](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-required-annotation) 注解可使属性成为必需的依赖
Spring 团队通常提倡构造器注入, 因为它使应用程序组件能够实现为不可变对象, 并确保所需的依赖项不为 `null`; 此外, 构造器注入的组件始终以完全初始化的状态返回到客户端 (调用) 代码; 另外, 大量的构造函数参数是一个糟糕的代码风格, 暗示该类可能有太多的责任, 应该重构以更好地解决关注点的正确分离
Setter 注入应主要仅用于可在类中指定合理默认值的可选依赖项; 否则, 必须在代码使用依赖项的任何位置执行非空检查; setter 注入的一个好处是 setter 方法使该类的对象可以在以后重新配置或重新注入; 因此, 通过 JMX MBean 进行管理是二次注入的一个令人激动的使用用例  
使用对特定类最有意义的 DI 样式; 有时, 在处理你没有源代码的第三方类时, 选择权在你; 例如, 如果第三方类没有公开任何 setter 方法, 那么构造函数注入可能是唯一可用的 DI 形式
```
##### 依赖解析过程
容器按照如下执行 bean 的依赖解析
- `ApplicationContext` 被创建并使用描述所有 beans 的配置元数据初始化; 配置元数据可以通过 XML, Java 代码或注解指定
- 对于每个 bean, 如果使用依赖于普通构造器的, 那么它的依赖关系将以属性, 构造器参数或 static-factory 方法的参数的形式表示; 实际创建 bean 时, 会将这些依赖提供给 bean
- 每个属性或者构造器参数都是设置值的一个真正定义, 或者容器中其它 bean 的一个引用
- 每个属性或构造器参数的值都从其指定格式转换为该属性或构造函数参数的实际类型; 默认情况下, Spring 可以将字符串格式提供的值转换为所有内置类型, 例如 `int`, `long`, `String`, `boolean` 等等

当容器被创建后, Spring 容器会校验每个 bean 的配置; 然而, bean 的属性在 bean 创建之前不会被设置; 当容器创建时, Bean 会创建为单例的并且设置为预先实例化的 (默认值)的; Scopes 在 [7.5 小节 "Bean 范围"](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes) 中定义; 否则, 则仅在请求时才创建 bean; 创建 bean 可能会导致 bean 的依赖图被创建, 因为 bean 的依赖及其依赖的依赖 (依此类推) 会被被创建和分配; 注意这些依赖之间的不匹配解析可能会较晚显现, 即在首次创建受影响的 bean 时
```
循环依赖
如果你主要使用构造器注入, 这可能造成一个不能解析的循环依赖场景  
例如, class A 通过构造器注入获取一个 class B 实例, 然而 class B 也通过构造器注入获取一个 class A 实例; 如果你为 class A 和 B 配置 bean 为相互注入, Spring IoC 容器在运行时会探测到这个循环依赖, 然后抛出 `BeanCurrentlyInCreationException`  
可能的解决方案是编辑源代码配置由 setter 而不是构造器配置; 即避免构造器注入并仅使用 setter 注入; 换句话说, 尽管不推荐使用, 但你可以使用 setter 注入配置循环依赖
与一般的情况 (没有循环依赖) 不同, bean A 和 sbean B之间的循环依赖强制其中一个 bean 在完全初始化之前被注入另一个bean (一个经典的鸡/蛋场景)
```
你通常可以相信 Spring 能正确的做事; 它在容器加载时检测配置问题, 例如对不存在的 bean 和循环依赖关系的引用; 当实际创建 bean 时, Spring 会尽可能晚地设置属性并解析依赖关系; 这意味着, Spring 容器可以在被正确加载之后, 在请求一个对象或其依赖有问题时抛出异常; 例如, bean 因缺少属性或无效属性而抛出异常; 这可能会延迟一些配置问题的可见性, 这就是默认情况下 `ApplicationContext` 实现预先实例化单例 bean 的原因; 以实际需要之前创建这些 bean 的一些前期时间和内存为代价，你会在创建 `ApplicationContext` 时发现配置问题, 而不是更晚; 你当然可以覆盖此默认行为, 以便将单例 bean 延迟初始化, 而不是预先实例化  
如果不存在循环依赖, 当一个或多个协作 bean 被注入依赖 bean 时, 每个协作 bean 在被注入依赖 bean 之前配置; 这意味着如果 bean A 依赖于 bean B, Spring IoC 容器在调用 bean A 上的 setter 方法之前会配置 bean B; 换句话说, bean 实例化 (如果不是预先实例化的单例) 依赖于设置和相关声明周期方法 (如 [配置的 init 方法](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-lifecycle-initializingbean) 或 [ InitializingBean 回调方法](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-factory-lifecycle-initializingbean)) 的调用

##### 依赖注入示例
以下示例基于 XML 的配置元数据用于基于 setter 的 DI; Spring XML 配置文件的一小部分指定了一些 bean 定义:
```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```
在以上的示例中, 声明 setter 与 XML 文件中指定的属性匹配; 以下示例使用基于构造器的 DI:
```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```
bean 定义中指定的构造器参数将会作为 `ExampleBean` 构造器的参数  
现在考虑此示例的一个变种, Spring 被告知调用一个静态工厂方法返回对象的一个实例, 而不是使用构造器方法
```
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```
静态工厂方法的参数通过 `<constructor-arg/>` 元素提供, 与使用的构造器完全相同; 工厂方法返回的类的类型不必与包含静态工厂方法的类具有相同的类型, 尽管在此示例中它是这样; 实例 (非静态) 工厂方法将以基本相同的方式使用 (除了使用 factory-bean 属性而不是 `class` 属性) 因此这里不再讨论细节。

#### 依赖与配置的细节
如上一节所述, 你可以将 bean 属性和构造器参数定义为对其他托管 bean (协作者) 的引用, 或者作为内联定义的值; 为此, Spring 基于 XML 的配置元数据支持其 `<property/>` 和 `<constructor-arg/>` 元素中的子元素类型

##### 字面值 (原生类, String 类等等)
`<property/>` 元素的 `value` 将属性或构造器参数指定为人类可读的字符串表示形式; Spring 的 [转换服务](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/validation.html#core-convert-ConversionService-API) 用于将这些值从 String 转换为属性或参数的实际类型
```
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```
为了更简洁的 XML 胚子以下示例使用了 [p-namespace](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-p-namespace)
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>

</beans>
```
前面的 XML 更简洁; 然而拼写错误会在运行时而不是设计时发现, 除非你在创建 bean 定义时使用支持自动属性完成的 IDEA 或 STS 等 IDE, 强烈建议使用此类 IDE 帮助  
你也可以如下配置一个 `java.util.Properties` 实例
```
<bean id="mappings"
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```
Spring 容器通过使用 JavaBeans 的 `PropertyEditor` 机制将 `<value/>` 元素内的文本转换为 `java.util.Properties` 实例; 这是一个很好的快捷方式, 并且是 Spring 团队支持在值属性样式上使用嵌套 `<value/>` 元素的地方之一

##### idref 元素
`idref` 元素只是一种防错方法, 可以将容器中另一个 bean 的 id (字符串值 - 而不是引用) 传递给 `<constructor-arg/>` 或 `<property/>` 元素
```
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```
以上 bean 定义的片段与以下片段 (在运行时) 相等
```
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```
第一种形式优于第二种形式, 因为使用 `idref` 标签允许容器在部署时验证引用的命名 bean 实际存在; 在第二个变体中, 不对传递给客户端 bean 的 `targetName` 属性的值执行验证; 当客户端 bean 实际被实例化时, 才会发现错别字 (最有可能致命的结果); 如果客户端 bean 是一个 [prototype](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes) bean，则只能在部署容器后很长时间才可能发现此错误和产生的异常
>在 4.0 beans xsd 中不再支持 `idref` 元素的 `local` 属性, 因为它不再提供常规 `bean` 引用的值; 升级到 4.0 架构时, 只需将现有的 `idref local` 引用更改为 `idref bean` 即可

`<idref/>` 元素带来值的常用于 (至少在 Spring 2.0 之前的版本中) 是在 ProxyFactoryBean bean 定义中的 AOP 拦截器的配置中; 指定拦截器名称时使用 `<idref/>` 元素可以防止拼写错误的拦截器 ID  

##### 引用其他 beans (协作者)
`ref` 元素是 `<constructor-arg/>` 或 `<property/>` 定义元素中的最后一个元素; 您可以将 bean 的指定属性的值设置为对容器管理的另一个 bean (协作者) 的引用; 引用的 bean 是 bean 的依赖项, 其属性将被设置, 并且在设置属性之前根据需要按需初始化; (如果协作者是单例 bean, 它可能已由容器初始化) 所有引用最终都是对另一个对象的引用; 范围和验证取决于您是否通过 `bean, local, parent` 属性指定其他对象的 id/name  
通过 `<ref/>` 标记的 bean 属性指定目标 bean 是最常用的形式, 并允许创建对同一容器或父容器中的任何 bean 的引用, 而不管它是否在同一 XML 文件中; bean 属性的值可以与目标 bean 的 id 属性相同, 也可以作为目标 bean 的 name 属性中的值之一
```
<ref bean="someBean"/>
```
通过 `parent` 属性指定目标 bean 会创建对当前容器的父容器中的 bean 的引用; `parent` 属性的值可以与目标 bean 的 id 属性相同, 也可以与目标 bean 的 name 属性中的一个值相同, 并且目标 bean 必须位于当前 bean 的父容器中; 您主要在拥有容器层次结构并且希望将现有 bean 包装在父容器中并使用与父 bean 具有相同名称的代理时使用此 bean 引用变体
```
<!-- in the parent context -->
<bean id="accountService" class="com.foo.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```
```
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```
>在 4.0 bean xsd 中不再支持 ref 元素的 local 属性, 因为它不再提供常规 bean 引用的值; 升级到 4.0 架构时, 只需将现有的 `ref local` 引用更改为 `ref bean`

##### 内部 beans
`<property/>` 或 `<constructor-arg/>` 元素中的 `<bean/>` 元素定义了一个所谓的内部 bean
```
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```
内部 bean 定义不需要定义的 id 或名称; 如果指定, 则容器不使用此类值作为标识符; 容器还会在创建时忽略范围标志: 内部 bean 始终是匿名的, 并且始终使用外部 bean 创建它们; 不能将内部 bean 注入协作 bean 而不是封闭 bean, 或者独立访问它们  
作为极端情况, 可以从自定义范围接收销毁回调; 例如, 对于包含在单例 bean 中的请求范围的内部 bean: 内部 bean 实例的创建将绑定到其包含的 bean, 但是销毁回调允许它参与请求范围的生命周期, 这不是常见的情况; 内部 bean 通常只是共享其包含 bean 的范围

##### 集合
在 `<list/>, <set/>, <map/>, <props/>` 元素中, 你可以分别设置 Java Collection 的 List，Set，Map, roperties 类型的属性和参数
```
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```
map 的键值或者 set 的值可以是以下任意一种类型
```
bean | ref | idref | list | set | map | props | value | null
```

##### 集合合并
Spring 容器还支持集合的合并; 应用程序开发人员可以定义父样式的 `<list/>, <map/>, <set/>, <props/>` 元素, 并具有子样式的 `<list/>, <map/>, <set/>, <props/>` 元素继承并覆盖父集合中的值; 也就是说, 子集合的元素覆盖父集合中指定的值, 子集合的值是合并父集合和子集合的元素的结果  
关于合并的这一部分讨论了父子 bean 机制, 不熟悉父子 bean 定义的读者可能希望在继续之前阅读 [相关部分](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-child-bean-definitions)  
以下示例演示了集合合并
```
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```
请注意在子 bean 定义的 `adminEmails` 属性的 `<props/>` 元素上使用了 `merge = true` 属性; 当容器解析并实例化子 bean 时, 生成的实例有一个 `adminEmails` 属性集合, 其中包含子项的 `adminEmails` 集合与父项的 `adminEmails` 集合的合并结果
```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```
子 Properties 集合的值集继承父 `<props/>` 的所有属性元素, 子值的支持值覆盖父集合中的值  
此合并行为同样适用于 `<list/>, <map/>, <set/>` 集合类型; 在 `<list/>` 元素的特定情况下, 保持与 List 集合类型相关联的语义, 即有序的值集合的概念; 父级的值位于所有子级列表的值之前; 对于 `Map, Set, Properties` 集合类型, 不存在排序; 因此, 对于作为容器内部使用的关联 `Map, Set, Properties` 实现类型的基础的集合类型, 没有排序语义生效

##### 集合合并的限制
你不能合并不同的集合类型 (例如 `Map` 和 `List`), 如果你尝试这样做, 则会抛出相应的 Exception; 必须在较低的继承子定义上指定 `merge` 属性; 在父集合定义上指定 `merge` 属性是多余的, 不会导致所需的合并

##### 强类型集合
在 Java 5中引入泛型类型, 你可以使用强类型集合; 也就是说, 可以声明 Collection 类型, 使其只能包含 `String` 元素; 如果你使用 Spring 依赖注入一个强类型的 Collection 到 bean 中, 你可以利用 Spring 的类型转换支持, 这样强类型 Collection 实例的元素在被添加到之前就会被转换为适当的类型集合
```
public class Foo {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```
```
<beans>
    <bean id="foo" class="x.y.Foo">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```
当 `foo` bean 的 `accounts` 属性准备好进行注入时, 可以通过反射获得有关强类型 `Map<Strin,Float>` 的元素类型的泛型信息; 因此, Spring 的类型转换基础结构将各种值元素识别为 `Float` 类型, 并将字符串值 `9.99, 2.75, 3.99` 转换为实际的 Float 类型

##### Null 和空字符串值
Spring 将属性等的空参数视为空字符; 以下基于 XML 的配置元数据片段将 email 属性设置为空 String 值 ("")
```
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```
以上示例等同于以下代码
```
exampleBean.setEmail("");
```
`<null/>` 元素处理 `null` 值, 例如
```
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```
以上示例等同于以下代码
```
exampleBean.setEmail(null);
```

##### 使用 p-namespace 的 XML 快捷方式
p-namespace 使你可以使用 bean 元素的属性而不是嵌套的 `<property/>` 元素来描述属性值或者协作 bean  
Spring 支持具有命名空间的可扩展配置格式, 这些命名空间基于 XML Schema 定义; 本章中讨论的 `bean` 配置格式在 XML Schema 文档中定义; 但是, p-namespace 未在 XSD 文件中定义, 仅存在于Spring的核心中  
以下示例显示了两个解析为相同结果的 XML 片段: 第一个使用标准 XML 格式, 第二个使用 p-namespace
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="foo@bar.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="foo@bar.com"/>
</beans>
```
该示例显示了 bean 定义中名为 email 的 p-namespace 中的属性; 这告诉 Spring 包含一个属性声明; 如前所述, p-namespace 没有 schema 定义, 因此你可以将属性的名称设置为属性名称  
下一个示例包括另外两个 `bean` 定义, 它们都引用了另一个 `bean`
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```
如你所见, 此示例不仅包含使用 p-namespace 的属性值, 还使用特殊格式来声明属性引用; 第一个 bean 定义使用 `<property name="spouse" ref="jane"/>` 来创建从 bean `john` 到 bean `jane` 的引用, 而第二个 bean 定义使用 `p:spouse-ref="jane"` 作为属性来完成相同的事情; 在这种情况下, `spouse` 是属性名称, 而 `-ref` 部分表示这不是直接值, 而是对另一个 bean 的引用
>p-namespace 不如标准 XML 格式灵活; 例如, 声明属性引用的格式与以 `Ref` 结尾的属性冲突, 而标准 XML 格式则不然; 我们建议你仔细选择你的方法并将其传达给你的团队成员, 以避免生成同时使用这三种方法的 XML 文档

##### 使用 c-namespace 的 XML 快捷方式
和 `使用 c-namespace 的 XML 快捷方式` 小节类似, c-namespace 在 Spring 3.1 中引入, 允许使用内联属性来配置构造函数参数, 而不是嵌套的 `constructor-arg` 元素  
让我们回顾一下带有 `c:namespace` 的基于构造函数的依赖注入一节中的示例
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bar" class="x.y.Bar"/>
    <bean id="baz" class="x.y.Baz"/>

    <!-- traditional declaration -->
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
        <constructor-arg value="foo@bar.com"/>
    </bean>

    <!-- c-namespace declaration -->
    <bean id="foo" class="x.y.Foo" c:bar-ref="bar" c:baz-ref="baz" c:email="foo@bar.com"/>

</beans>
```
`c:namespace` 名称空间使用与 `p:namespace` (用于 bean 引用的trailing `-ref`) 有相同的约定, 用于按名称设置构造函数参数; 同样, 它需要声明, 即使它没有在 XSD 中定义 (但它存在于 Spring Core 中)

TODO

##### 复合属性名称
只要除最终属性名称之外的路径的所有组件都不为 `null`, 你可以在设置 bean 属性时使用复合或嵌套属性名称; 见以下 bean 定义
```
<bean id="foo" class="foo.Bar">
    <property name="fred.bob.sammy" value="123" />
</bean>
```
`foo` bean 有一个 `fred` 属性, 它有一个 `bob` 属性, 它有一个 `sammy` 属性, 最后的 `sammy` 属性被设置为值 `123`; 为了使之工作, `foo` 的 `fred` 属性和 `bob` 属性在构造 bean 之后, `fred` 不能为 `null`, 否则抛出 `NullPointerException`

##### 使用 depends-on
如果 bean 是另一个 bean 的依赖项, 通常意味着将一个 bean 设置为另一个 bean 的属性; 通常你可以使用基于 XML 的配置元数据中的 `<ref/>` 元素来完成此操作; 但有时 bean 之间的依赖关系不那么直接; 例如, 需要触发类中的静态初始化程序, 例如数据库驱动程序注册; 在初始化使用此元素的 bean 之前, depends-on 属性可以显式强制初始化一个或多个 bean; 以下示例使用 depends-on 属性表示对单个 bean 的依赖关系:
```
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```
>bean 定义中的 `depends-on` 属性既可以指定初始化时间依赖关系, 也可以指定仅限单例 bean 的相应销毁时间依赖关系; 在给定的 bean 本身被销毁之前, 首先销毁定义与给定 bean 的依赖关系的从属 bean; 因此, 依赖也可以控制销毁顺序

##### 延迟初始化的 beans
默认情况下, ApplicationContext 实现会激进地创建和配置所有单例 bean, 作为初始化过程的一部分; 通常, 这种预先实例化是可取的, 因为配置或周围环境中的错误是立即发现的, 而不是几小时甚至几天后; 如果不希望出现这种情况, 可以通过将 bean 定义标记为延迟初始化来阻止单例 bean 的预实例化; 延迟初始化的 bean 告诉 IoC 容器在第一次请求时创建 bean 实例, 而不是在启动时  
在 XML 中，此行为由 `<bean/>` 元素上的 `lazy-init`属性控制; 例如
```
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.foo.AnotherBean"/>
```
当 ApplicationContext 使用上述配置时, 在 ApplicationContext 启动时不会激进地预先实例化名为 `lazy` 的 bean, 然而会激进地预先实例化 `not.lazy` bean  
但是, 当延迟初始化的 bean 是未进行延迟初始化的单例 bean 的依赖项时, ApplicationContext 会在启动时创建延迟初始化的 bean, 因为它必须满足单例的依赖关系; 延迟初始化的 bean 被注入到其他地方的单例 bean 中, 而这个 bean 并不是延迟初始化的  
你可以使用 `<beans/>` 元素上的 `default-lazy-init` 属性控制容器级别的延迟初始化; 例如
```
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

##### 自动装配协作者
Spring 容器可以自动连接协作 bean 之间的关系; 你可以通过检查 ApplicationContext 的内容, 允许 Spring 自动为你的 bean 解析协作者 (其他bean); 自动装配具有以下优点
- 自动装配可以显着减少指定属性或构造函数参数的需要 (在本章其他地方讨论的其他机制, 如 bean 模板, 在这方面也很有价值)
- 自动装配可以随着对象的发展更新配置; 例如, 如果需要向类添加依赖项, 则可以自动满足该依赖项, 而无需修改配置; 因此, 自动装配在开发期间尤其有用, 而不拒绝在代码库变得更稳定时切换到显式布线的选择

使用基于 XML 的配置元数据时, 可以使用 `<bean/>` 元素的 `autowire` 属性为 bean 的定义指定 autowire 模式; 自动装配功能有四种模式, 你可以为每个 bean 都指定自动装配, 因此需选择那些自动装配的选项

| 模式 | 解释     |
| :--- | :--- |
| no | (默认) 无自动装配; 必须通过 `ref` 元素定义 Bean 引用; 不建议对较大的部署更改默认设置, 因为明确指定协作者可以提供更好的控制和清晰度; 在某种程度上, 它记录了系统的结构 |
| byName | 按属性名称自动装配; Spring 查找与需要自动装配的属性同名的 bean; 例如, 如果 bean 定义按名称设置为 autowire, 并且它包含 master 属性 (即它具有 setMaster(..) 方法), 则 Spring 会查找名为 master 的 bean 定义, 并使用它来设置属性 |
| byType | 如果容器中只存在一个属性类型的 bean, 则允许自动装配属性; 如果存在多个, 则抛出异常, 这表示你不能对该 bean 使用 byType 自动装配; 如果没有匹配的 bean, 什么都不会发生, 此属性也不会被设置 |
| constructor | 类似于 byType, 但适用于构造函数参数; 如果容器中没有构造函数参数类型的一个 bean, 则会引发错误 |

使用 byType 或构造函数自动装配模式, 你可以连接数组和类型集合; 在这种情况下, 提供容器内与预期类型匹配的所有 autowire 候选者以满足依赖性; 如果预期的键类型为 `String`, 则可以自动装配强类型映射; 自动装配的 Maps 值将包含与预期类型匹配的所有 Bean 实例, 而 Maps 键将包含相应的 bean 名称  
你可以将 autowire 行为与依赖性检查相结合, 这将在自动装配完成后执行
###### 自动装配的局限和不足
当在整个项目中一致地使用自动装配时效果最佳; 如果一般不使用自动装配, 那么开发人员使用它来连接一个或两个 bean 定义可能会让人感到困惑  
考虑自动装配的局限和缺点:
- property 和 constructor-arg 设置中的显式依赖项始终覆盖自动装配; 你无法自动装配所谓的简单属性, 例如原始类型, `String` 和 `Classes` (以及此类简单属性的数组); 这种限制是有意设计的
- 自动装配不如显式布线精确; 尽管如上表所示, Spring 会小心避免在可能产生意外结果的歧义的情况下进行猜测, 但不再明确记录 Spring 管理对象之间的关系
- 可能无法为可能从 Spring 容器生成文档的工具提供接线信息
- 容器中的多个 bean 定义可以匹配 setter 方法或构造函数参数指定的类型以进行自动装配; 对于数组, 集合或地图, 这不一定是个问题; 但是, 对于期望单个值的依赖关系, 这种模糊性不是任意解决的; 如果没有可用的唯一 bean 定义, 则抛出异常。

在最后的场景中, 你有以下几种选择
- 放弃自动装配，支持显式布线
- 通过将 `autowire-candidate` 属性设置为 `false`, 避免对 bean 定义进行自动装配, 如下一节所述
- 通过将其 `<bean/>` 元素的 `primary` 属性设置为 `true`, 将单个 `bean` 定义指定为主要候选者
- 使用基于注释的配置实现更细粒度的控制, 见 [第 7.9 节 "基于注释的容器配置" 中所述](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-annotation-config)
###### 从自动装配中排除一个 bean
TODO

##### 方法注入
在大多数应用程序场景中, 容器中的大多数 bean 都是单例; 当单例 bean 需要与另一个单例 bean 协作, 或者非单例 bean 需要与另一个非单例 bean 协作时, 通常通过将一个 bean 定义为另一个 bean 的属性来处理依赖关系; 当 bean 生命周期不同时会出现问题; 假设单例 bean A 需要使用非单例 (原型) bean B, 可能是在 A 上的每个方法调用上; 容器只创建一次单例 bean A, 因此只有一次机会来设置属性; 每次需要时, 容器都不能为 bean A 提供 bean B 的新实例  
解决方案是放弃一些控制反转; 你可以通过实现 `ApplicationContextAware` 接口 [使 bean A 了解容器](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-aware), 并通过 [对容器进行 getBean("B") 调用](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-client), 每次 bean A 需要时都要求 (通常是新的) bean B 实例; 以下是此方法的示例
```
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```
前面的内容是不可取的, 因为业务代码知道并耦合到 Spring Framework; Method Injection 是 Spring IoC 容器的一个高级功能, 它允许以干净的方式处理这个用例  
你可以在 [此博客](https://spring.io/blog/2004/08/06/method-injection/) 中阅读有关 Method Injection 的动机的更多信息
###### Lookup 方法注入
Lookup 方法注入是容器覆盖容器托管 bean 上方法的能力, 以返回容器中另一个命名 bean 的查找结果; 查找通常涉及原型 bean, 如上一节中描述的场景; Spring Framework 通过使用 CGLIB 库中的字节码生成来实现此方法注入, 以动态生成覆盖该方法的子类
>- 为了使这个动态子类化工作, Spring bean 容器将子类化的类不能是 `final`, 并且要重写的方法也不能是 `final`
>- 对具有抽象方法的类进行单元测试需要你自己对类进行子类化, 并提供抽象方法的存根实现
>- 组件扫描也需要具体的方法, 这需要具体的类别来获取
>- 另一个关键限制是查找方法不适用于工厂方法, 特别是配置类中的 `@Bean` 方法, 因为容器在这种情况下不负责创建实例, 因此无法动态的创建运行时生成的子类

查看前面代码片段中的 `CommandManager`类, 你会看到 Spring 容器将动态覆盖 createCommand() 方法的实现; 你的 `CommandManager` 类将没有任何 Spring 依赖项, 如在重新编写的示例中可以看到的那样
```
package fiona.apple;

// no more Spring imports!

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
在包含要注入的方法的客户端类 (在本例中为 CommandManager)中, 要注入的方法需要以下形式的签名
```
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```
如果方法是抽象的, 则动态生成的子类实现该方法; 否则, 动态生成的子类将覆盖原始类中定义的具体方法; 例如
```
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```
标识为 commandManager 的 bean 在需要 myCommand bean 的新实例时调用自己的方法 createCommand(); 你必须小心将 myCommand bean 部署为原型, 如果这实际上是需要的话; 如果它是一个单例, 则每次都返回 myCommand bean 的相同实例  
可选的, 在基于注释的组件模型中, 你可以通过 `@Lookup` 注解声明查找方法
```
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```
或者是更常见的形式, 你可以依赖于针对查找方法的声明返回类型来解析目标 bean
```
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```
请注意,你通常会使用具体的存根实现声明这种带注释的查找方法, 以使它们与 Spring 的组件扫描规则兼容, 默认情况下抽象类被忽略; 此限制不适用于显式注册或显式导入的 bean 类
>访问不同范围的目标 bean 的另一种方法是 ObjectFactory/Provider 注入点; 见 ["Scope bean 作为依赖" 小节](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-other-injection); 感兴趣的读者也可以找到 `ServiceLocatorFactoryBean`(在 org.springframework.beans.factory.config 包中)

###### 任意方法替换
与查找方法注入相比, 一种不太有用的方法注入形式是能够使用另一个方法实现替换托管 bean 中的任意方法; 用户可以安全地跳过本节的其余部分, 直到实际需要该功能  
使用基于 XML 的配置元数据, 你可以使用被替换的方法元素将已部署的 bean 的方法实现替换; 考虑以下类, 我们要覆盖方法 `computeValue`
```
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```
实现 `org.springframework.beans.factory.support.MethodReplacer` 接口的类可以提供新方法定义
```
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```
部署原始类并指定方法覆盖的 bean 定义如下所示
```
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```
你可以在 `<replacement-method/>` 元素中使用一个或多个包含的 `<arg-type/>` 元素来指示被覆盖的方法的方法签名; 仅当方法重载且类中存在多个变体时才需要参数的签名; 为方便起见, 参数的类型字符串可以是完全限定类型名称的子字符串;  例如, 以下所有内容都匹配 `java.lang.String`
```
java.lang.String
String
Str
```
因为参数的数量通常足以区分每个可能的选择, 所以通过允许你只键入与参数类型匹配的最短字符串, 此快捷方式可以节省大量的输入

>**参考:**  
[Dependencies](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-dependencies)
