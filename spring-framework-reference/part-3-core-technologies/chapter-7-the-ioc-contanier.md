### IoC 容器

#### Spring IoC 容器和 beans 介绍
本章涵盖了 Spring 框架的控制反转实现 (IoC) 原理; IoC 也被称为依赖注入 (DI); 这是一个定义它们依赖的过程, 即它们使用的其他对象, 仅通过构造函数参数, 工厂方法的参数, 或者在构造或从工厂方法返回后在对象实例上设置的属性; 然后容器在创建 bean 时注入这些依赖; 这个过程基本上是反向的, 因此命名为控制反转 (IoC), bean 本身通过直接使用类构建器或者类似 Service Locator 模式的机制来控制依赖的实例化和位置  
`org.springframework.beans` 和 `org.springframework.context` 包是 Spring 框架 IoC 容器的基础; `BeanFactory` 接口提供了能够管理任何类型对象的高级可配置机制; `ApplicationContext` 是 `BeanFactory` 的一个子接口; 它添加了更易与 Spring AOP 功能的集成; 消息资源处理 (为了用于国际化), 事件发布; 以及例如用于 web 应用的 `WebApplicationContext` 的应用层指定上下文  
简而言之, ·`BeanFactory` 提供了配置框架和基础功能, `ApplicationContext` 添加了更多企业级的功能; `ApplicationContext` 是 `BeanFactory` 的一个完整超集, 在本章中仅用于 Spring IoC 容器的描述; 更多关于使用 `BeanFactory` 而不是 `ApplicationContext` 的信息, 参见 [7.16 小节, BeanFactory](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-beanfactory)  
在 Spring 中, 构成你应用程序的主干的并且被 Spring Ioc 容器管理的对象称为 beans; 一个 bean 是一个被实例化的, 装备过的, 并且被 Sprign IoC 容器管理的对象, bean 是你应用程序中众多对象中的一个; Bean 及其之间的依赖关系反映在容器使用的配置元数据中

#### 容器概览
接口 `org.springframework.context.ApplicationContext` 表示 Spring IoC 容器, 并为负责实例化, 配置, 装配上述提及的 beans;  容器通过读取配置元数据获取对象实例化, 配置, 以及装配指令; 配置元数据可使用 XML, Java 注解, 或者 Java 代码表示; 它允许你表示组成你的应用程序的对象和这些对象之间的丰富的依赖性  
Spring 提供了几个开箱即用的 `ApplicationContext` 接口的实现; 在独立程序中通常创建一个 `ClassPathXmlApplicationContext` 或者 `FileSystemXmlApplicationContext` 的实例; 虽然 XML 一直是定义配置元数据的传统格式, 但你可以通过提供少量 XML 配置来声明性地支持这些额外的元数据格式, 从而指示容器使用 Java 注解或代码作为元数据格式  
在大多数的应用场景中, 不需要显示的用户代码来实例化一个或多个 Spring IoC 容器的实例; 例如, 在 web 应用场景中, 应用程序的 web.xml 文件中的简单八行 (左右) 样板 Web 描述符 XML 通常就足够了 (见 [7.15.4 小节, web 应用程序便捷的 ApplicationContext 实例化](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#context-create)); 如果你使用的是 Spring Tool Suite Eclipse 支持的开发环境, 只需点击几下鼠标或按键即可轻松创建此样板文件配置  
下图是 Spring 如果工作的高度抽象; 你的应用程序类与配置元数据相结合, 以便随后 `ApplicationContext` 的创建和实例化, 你拥有一个完全配置且可执行的系统或应用程序  
![The Spring IoC container](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/images/container-magic.png)  
##### 配置元数据
如上图所示, Spring IoC 容器使用一种形式的配置元数据; 这个配置元数据表示你作为一个开发者如何告诉 Spring 容器去实例化, 配置, 以及装配你的应用程序中的对象  
配置元数据通常以简单直观的 XML 格式提供, 本章大部分内容用此来传达 Spring IoC 容器的关键概念和功能  
> 基于 XML 的元数据并不是唯一允许的配置元数据的形式; Spring IoC 容器与实际的配置元数据书写形式是完全分离的; 最近许多开发者为他们的 Spring 应用程序选择 [基于 Java 的配置](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-java)

Spring 容器使用其他形式的元数据的更多信息见
- [基于注解的配置](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-annotation-config): Spring 2.5 引入了对基于注解配置元数据的支持
- [基于 Java 的配置](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-java): 从 Spring 3.0开始, Spring JavaConfig 详细提供的许多功能成为了 Spring 框架核心的一部分; 因此你可以在你应用程序之外使用 Java 而不是 XML 文件定义 beans; 为了使用这些新功能, 见 `@Configuration`, `@Bean`, `@Import` 和 `@DependsOn` 注解  

Spring 配置由至少一个, 通常是多于一个的容器管理的 bean 的定义组成; 基于 XML 配置的元数据, 这些 beans 配置是作为顶层 `<beans/>` 标签中的 `<bean/>` 元素表示的; Java 配置通常是在 `@Configuration` 注解的类中的使用 `@Bean` 注解的方法表示  
这些对应这实际对象的 bean 定义组成了你的应用程序; 通常你会定义服务层对象, 数据访问对象 (DAOs), 例如 Struts `Action` 实例的表现层对象, 例如 Hibernate `SessionFactories`, JMS `Queues` 的基础架构对象等等; 通常不会配置细粒度的领域对象在容器中, 因为这通常由 DAOs 和业务逻辑负责创建和加载领域对象; 然而, 你可以使用 Spring 整合 AspectJ 去配置在 IoC 容器控制之外创建的对象; 见 [在 Spring 中使用 AspectJ 去依赖注入领域对象](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/aop.html#aop-atconfigurable)  
以下实例展示了基于 XML 配置元数据的基本结构
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```
`id` 属性是你用于标识个体 bean 定义的一个字符串; `class` 属性使用全限定名定义了这个 bean 的类型; id 属性的值可以被协作对象引用; 在 XML 中对于协作对象的引用没有展示在以上实例中; 更多信息见 [依赖](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-dependencies)

##### 实例化一个容器
实例化一个 Spring IoC 容器是直接了当的; 提供给 ApplicationContext 构造函数的位置路径实际上是资源字符串, 允许容器从各种外部资源 (如本地文件系统, Java CLASSPATH 等) 加载配置元数据  
```
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```
>在你学习了 Spring IoC 容器之后, 你可能想知道更多关于 Spring `Resource` 抽象的信息, 在 [第 8 章 Resources](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/resources.html) 叙述, 它对于从定义在 URI 语法中的地址读取一个 InputStream 流提供了一种便捷的机制, `Resource` 路径用于构建应用程序上下文在 [8.7 小节, 应用程序上下文和资源路径](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/resources.html#resources-app-ctx) 中叙述

以下实例展示了服务层对象 (`services.xml`) 的配置文件
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```
以下实例展示了数据访问对象 (`daos.xml`) 的配置文件
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```
在前面的示例中, 服务层由 PetStoreServiceImpl 类和两个 JpaAccountDao 和 JpaItemDao 类型的数据访问对象组成 (基于JPA对象/关系映射标准); 属性 name 元素引用 JavaBean 属性的名称, ref 元素引用另一个 bean 定义的名称; id 和 ref 元素之间的这种联系表达了协作对象之间的依赖关系; 有关配置对象的依赖关系的详细信息, 见 [依赖](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-dependencies)

###### 组成基于 XML 的配置元数据
让 bean 定义跨越多个 XML 文件会很有用, 通常每个 XML 配置文件都代表架构中的一个逻辑层或模块  
你可以使用 ApplicationContext 构造器从所有的这些 XML 片段中加载 bean 定义; 构造器会使用多个 `Resource` 位置, 如上一节展示的那样; 可选的, 可以使用一个或多个 `<import/>` 元素从其他文件加载 bean 定义; 例如
```
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```
在以上示例中, 外部的 bean 定义会从三个文件中加载: `services.xml`, `messageSource.xml`, `themeSource.xml`; 所有的位置路径是相对于执行导入的定义文件, 所以 `services.xml` 一定和执行导入的文件在同一个目录或者类路径位置, 而 `messageSource.xml`, `themeSource.xml` 一定在与执行导入文件
下方的一个 `resources` 位置; 正如你所见, 打头的斜杠会被忽略, 因为给定的路径是相对的, 做好的形式完全不要使用斜杠; 文件的内容将会被导入, 包含在顶级标签 `<beans/>` 元素中, 根据 Spring Schema 必须是有效的 XML bean 定义  
>这是可能的, 但是并不推荐, 使用 "../" 指定父目录中的引用文件; 这样做会创建一个在当前应用程序之外文件的一个依赖; 特别的, 此引用不推荐使用 "classpath:" 的 URLs (例如, "classpath:../services.xml"), 运行时解析过程选择 "最近的" 类路径根查看其父目录; Classpath 配置的改变会导致不同的选择, 不正确的目录; 你可以总是使用全限定资源路径代替相对路径: 例如, "file:C:/config/services.xml" 或 "classpath:/config/services.xml"; 然而, 要意识到你这时在将你应用程序的配置绑定到一个指定的绝对位置; 通常最好为这样的绝对位置保持间接, 例如通过在运行时针对 JVM 系统属性解析的 "$ {...}" 占位符

import 指令是 beans 命名空间自己提供的功能;
172/5000
除了普通 bean 定义之外的其他配置功能在 Spring 提供的一系列 XML 命名空间中可用, 例如: "context" 和 "util" 命名空间

###### Groovy Bean 定义 DSL
作为外化配置元数据的另一个示例, bean 定义也可以在 Spring 的 Groovy Bean 定义 DSL 中表示, 如 Grails 框架; 通常, 此类配置将存在 ".groovy" 文件中,其结构如下:
```
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```
此配置样式在很大程度上等同于 XML bean 定义, 甚至支持 Spring 的 XML 配置命名空间; 它还允许通过 "importBeans" 指令导入 XML bean 定义文件

##### 使用容器
`ApplicationContext` 是高级工厂接口, 可以维护不同 beans 的注册以及它们的依赖; 你可以使用 `T getBean(String name, Class<T> requiredType)` 方法搜索你的 beans 实例  
`ApplicationContext` 能够使你读取 bean 定义以及访问它们:
```
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```
使用 Groovy 配置，bootstrapping 看起来非常相似, 只是一个不同的上下文实现类, 它是 Groovy 感知的 (也可以理解 XML bean 定义):
```
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```
最灵活的变量是 `GenericApplicationContext` 联合一个代理读取器, 例如对于 XML 文件的 `XmlBeanDefinitionReader`
```
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```
对于 Groovy 文件使用 `GroovyBeanDefinitionReader`
```
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```
这些读取器可以委托在同一个 ApplicationContext 上混合使用, 如果需要可以从不同的配置源读取 bean 定义  
你可以使用 `getBean` 来所有你的 beans 实例; `ApplicationContext` 接口有一些其他的接口来搜索这些 beans, 但最好你的程序代码从不使用它们; 事实上, 你的程序代码应该完全不调用 `getBean()` 方法, 这样就完全不依赖于 Spring APIs; 例如, Spring 与 Web 框架的集成为各种 Web 框架组件 (如控制器和 JSF 托管 bean) 提供依赖注入, 允许你通过元数据 (例如自动装配注解) 声明对特定 bean 的依赖

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

#### 依赖
