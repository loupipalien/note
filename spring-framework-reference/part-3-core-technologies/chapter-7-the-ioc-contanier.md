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
>这是可能的, 但是并不推荐, 使用 "../" 指定父目录中的引用文件
