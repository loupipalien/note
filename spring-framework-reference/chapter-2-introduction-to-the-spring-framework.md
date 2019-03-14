### Spring 框架介绍
Spring 框架是一个为开发 Java 应用提供全面基础支持的 Java 平台; Spring 处理基础架构, 这样你可以专注你的应用程序  
Spring 使你可以从 plain old Java objects (POJOs) 构建应用程序, 并且可以无侵入的将企业服务应用于 POJOs;
这些功能可以应用于 Java SE 编程模型以及全部或部分的 Java EE  
作为一名应用程序开发者, 你可以从 Spring 平台收益示例如下:
- 使一个在数据库事务中执行的 Java 方法不用处理事务的 APIs
- 使一个本地 Java 方法的 HTTP 端不用处理 Servlet API
- 使一个本地 Java 方法的消息处理器不用处理 JMS API
- 使一个本地 Java 方法的管理操作不用处理 JMX API

#### 依赖注入和控制反转
Java应用程序 - 从受约束的嵌入式应用程序到n层服务器端企业应用程序的宽松术语 - 通常由协作形成应用程序的对象组成; 因此, 应用程序中的对象彼此依赖  
虽然 Java 平台提供了丰富的应用程序开发功能, 但是它缺少将基本构建块组织成连贯整体的方法, 而将这项任务留给了架构师和开发者; 虽然你可以使用例如工厂, 抽象工厂, 建造者, 装饰器, 服务定位器等设计模式组合各种类和对象实例来组成一个应用程序, 这些模式简单来说是: 给予最佳实践的一个名称, 描述这些模式的作用, 它应用的位置, 它解决的问题, 等等; 模式是在你的应用程序中你必须实现的规范化的最佳实践  
Spring 框架控制反转 (IoC) 组件解决这个问题是通过提供一种将不同组件组合成一个完全可以使用的应用程序的规范化方法; Spring 框架将编写规范化的设计模式作为可以集成到你自己的应用程序中的头等对象; 许多组织和机构以这种方式使用 Spring 框架来构建健壮的, 可维护的应用程序
>**背景**
"问题是, [它们] 是控制哪个方面的反转?"; 2004 年, Martin Fowler 在 [自己的网站](http://martinfowler.com/articles/injection.html) 上提出的关于控制反转的这个问题; Fowler 建议重命名这个原则使它更具有自解释性并且提出了注入依赖

#### 框架模块
Spring 框架由大约由 20 多个模块的功能组成; 这些模块分为 Core Container, Data Access/Integration, Web, AOP (Aspect Oriented Programming), Instrumentation, Messaging, and Test, 如下图所示
![Overview of the Spring Framework](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/images/spring-overview.png)
以下章节列出了每个功能的可用模块及其构件名称及其涵盖的主题; 构件名称与依赖管理工具中的构建 ID 相关联

##### 核心容器
[核心容器](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-introduction) 由 `spring-core`, `spring-beans`, `spring-context`, `spring-context-support`, 和 `spring-expression` (Spring Expression Language) 模块组成  
`spring-core` 和 `spring-beans` 模块提供了 [框架的基础部分](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-introduction), 包括 IoC 和依赖注入功能; `BeanFactory` 是工厂模式的一个复杂实现; 它消除了对编程单例的需求, 并允许你从你的实际程序逻辑中分离出依赖的配置的规范  
[Context](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#context-introduction) (`spring-context`) 模块构建于 [Core 和 Beans](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/beans.html#beans-introduction) 模块提供的稳固的基础之上: 它是一个类似与 JNDI 注册的使用框架风格形式访问对象的方法; Context 模块从 Beans 模块继承了它的功能并添加了对国际化的支持 (例如, 使用资源绑定), 事件传播, 资源加载, 以及透明的上下文创建, 例如 Servlet 容器; Context 模块也支持例如 EJB, JMX, 以及基本的远程等 Java EE 功能; `ApplicationContext` 接口是 Context 模块的重点; `spring-context-support` 提供了对通用的第三方库集成到 Spring 应用程序上下文的中支持, 以进行缓存 (EhCache, Guava, JCache), 邮件 (JavaMail), 调度(CommonJ, Quartz) 以及模板引擎 (FreeMarker, JasperReports, Velocity)  
`spring-expression` 模块提供了一个强大的 [Expression Language](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/expressions.html), 用于在运行时查询和操纵对象图; 它是 JSP 2.1 规范中指定的统一表达式语言 (统一的 EL) 的扩; 此语言支持属性值的设置与获取, 属性分配, 方法调用, 数组, 集合, 索引内容的访问, 逻辑和算术操作符, 命名变量, 从 Spring IoC 容器中按名称搜索对象; 它还支持列表投影和选择以及常用的列表聚合

##### AOP 和工具
`spring-aop` 模块提供了 [AOP](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/aop.html#aop-introduction) Alliance 兼容的面向切面编程的实现, 允许你定义, 例如方法拦截器和切入点, 以干净地解耦实现应该分离的功能的代码; 使用源码级别的元数据功能, 你可以将行为信息合并到你的代码中, 以一种类似于 .NET 属性的方式  
单独的模块 `spring-aspects` 提供了与 AspectJ 的整合  
`spring-instrument` 模块提供了用于特定应用服务器的类工具支持和类加载器实现; `spring-instrument-tomcat` 模块包含对 Tomcat 的 Spring 工具代理

##### 消息发送
Spring 框架 4 包括一个 `spring-messaging` 模块, 其中包含来自 Spring Integration 项目的关键抽象, 例如 `Message`, `MessageChannel`, `MessageHandler`, 以及其他为基于消息应用程序服务的基础; 此模块也包含用于映射消息到方法的一系列注解, 类似于基于编程模型的 Spring MVC 的注解

##### 数据访问/整合
数据访问/整合层由 JDBC, ORM, JMS 和事务模块组成  
`spring-jdbc` 模块提供了 JDBC 抽象层, 移除了烦杂的 JDBC 编码的需要并且解析数据库供应商指定的错误码  
`spring-tx` 模块支持对实现特殊接口的类和你所有的 POJOs (Plain Old Java Objects) 进行 [编码和声明式事务](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/transaction.html) 管理  
`spring-orm` 模块为流行的 [对象关系映射](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/orm.html#orm-introduction) APIs 提供了整合层, 包括 [JPA](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/orm.html#orm-jpa), [JDO](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/orm.html#orm-jdo) 和 [Hibernate](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/orm.html#orm-hibernate); 使用 `spring-orm` 模块你可以联合 Spring 提供的其他功能使用所有的 O/R-映射框架, 例如之前提到的生民式事务管理功能  
`spring-oxm` 模块提供了支持例如 JAXB, Castor, XMLBeans, JiBX, XStream 等 [Object/XML 映射](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/oxm.html) 实现的抽象层  
`spring-jms`模块 ([Java Messaging Service](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/jms.html)) 包含消息产生和消费的功能; 自 Spring 框架 4.1, 它提供了与 `spring-messaging` 模块的整合

##### Web
Web 层由 `spring-web`, `spring-webmvc`, `spring-websocket`, `spring-webmvc-portlet` 模块组成  
`spring-web` 模块提供了基础的面向 web 整合功能, 例如多部件文件上传功能和使用 Servlet 监听器和面向 web 应用的上下文初始化 IoC 容器; 它也包含一个 HTTP 客户端和 Spring 远程支持的 web 相关部分  
`spring-webmvc` 模块 (也被称作 Web-Servlet 模块) 包含对 web 应用程序的 Spring 的模型 - 视图 - 控制器 (MVC) 和 REST Web Service 实现; Spring 的 MVC 框架在领域模型代码和 web 表单之间提供了干净的分离, 并且整合了 Spring 框架的其他功能  
`spring-webmvc-portlet` 模块 (也被称作 Web-Portlet 模块) 提供了在Portlet环境中使用的 MVC 实现，并镜像了基于 Servlet 的 spring-webmvc 模块的功能

##### 测试
`spring-test` 模块提供了 [单元测试](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/unit-testing.html) 以及与 JUnit 或 TestNG 的 Spring 组件的 [整合测试](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/integration-testing.html); 它提供了 Spring ApplicationContexts 的一致性加载和这些上下文的缓存; 它还提供了你可用于独立测试你的代码的模拟对象

#### 使用场景
前面描述的构建块使 Spring 成为许多场景中的合理选择, 从在资源受限设备上运行的嵌入式应用程序到使用 Spring 的事务管理功能和 Web 框架集成的成熟企业应用程序
![Typical full-fledged Spring web application](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/images/overview-full.png)
Spring 的 [声明式事务管理功能](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/transaction.html#transaction-declarative) 使 Web 应用程序完全是事务性的, 就像使用 EJB 容器管理的事务一样; 你可以使用简单的 POJO 实现所有自定义业务逻辑, 并由 Spring 的 IoC 容器管理; 其他服务包括对发送邮件和校验的支持是独立于 Web 层的, 这使得你可以选择执行校验规则的位置; Spring 的 ORM 支持与 JPA, Hibernate, JDO 集成在一起; 例如, 在使用 Hibernate 时, 你可以继续使用现有的映射文件和标准的 Hibernate `SessionFactory` 配置; 表单控制器将 Web 层与域模型无缝集成, 无需 ActionForms 或其他将 HTTP 参数转换为域模型值的类
![Spring middle-tier using a third-party web framework](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/images/overview-thirdparty-web.png)
509/5000
有时情况不允许你完全切换到不同的框架; Spring Framework 不会强迫你使用其中的所有内容; 它不是一个全有或全无的解决方案; 使用 Struts, Tapestry, JSF 或其他 UI 框架构建的现有前端可以与基于 Spring 的中间层集成, 从而允许你使用 Spring 事务功能; 你只需使用 ApplicationContext 连接业务逻辑, 并使用 WebApplicationContext 集成 Web 层
![Remoting usage scenario](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/images/overview-remoting.png)
当你需要通过 Web 服务访问现有代码时, 可以使用 Spring 的 `Hessian-`, `Burlap-`, `Rmi-` 或 `JaxRpcProxyFactory` 类; 启用对现有应用程序的远程访问并不困难
![EJBs - Wrapping existing POJOs](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/images/overview-ejb.png)
Spring Framework 还为 Enterprise JavaBeans 提供了一个访问和抽象层, 使你能够重用现有的 POJO 并将它们包装在无状态会话 Bean 中, 以便在可能需要声明性安全性的, 可伸缩的, 故障安全 Web 应用程序中使用

##### 依赖管理和命名规约
依赖管理和依赖注入是不同的事; 为了在你的应用程序中基础 Spring 中的优秀功能 (如依赖注入), 你需要组装所需的库 (jar 文件) 并且将它们放在运行时的 classpath 中, 也可能是编译时; 这些依赖项不是注入的虚拟组件, 而是文件系统中的物理资源 (通常的); 依赖管理的处理设计定位这些资源, 存储它们, 并且将它们添加到 classpath 中; 依赖可以是直接的 (例如: 我的应用程序依赖 Spring), 或者是间接的 (例如: 我的应用程序依赖于依赖 `common-pool` 的 `common-dbcp`); 间接依赖也被称为传递性, 并且这些依赖也是最难识别和管理的  
如果你打算使用 Spring, 那么你需要获取你需要的 Spring 部分的 jar 库副本; 为了使这更容易, Spring 尽可能的分离依赖打包成一系列的模块, 例如你不需要 spring-web 模块如果你不想写一个 web 应用程序; 在本指南中为了指明 Spring 库, 我们使用一个简写的命名规约 `spring-*` 或者 `spring-*.jar`, `*` 代表着模块的简短名称 (例如: `spring-core`, `spring-webmvc`, `spring-jms` 等); 你使用的实际 jar 文件名通常是与版本号连接的模块名称 (例如: spring-core-4.3.20.RELEASE.jar)  
Spring 框架的每个发行版都会推送构件到以下地址:
- Maven 中央库, 这是 Maven 查询的默认仓库, 并且不要需要指定配置就可以使用; Spring 依赖的大多数公共库也可从 Maven 中央库获得, 并且 Spring 社区的很大一部分使用 Maven 作为依赖管理, 因为这对他们来说很方便; 这些 jars 名称形如 `spring-*-<version>.jar` 并且 Maven 的组标识是 `org.springframework`  
- 在专门为 Spring 托管的公共 Maven 仓库中; 除了最终的 GA 发行版, 这个仓库也托管了开发快照版和里程碑版; jar 文件命名与 Maven 中央库的形式相同, 这里的一个有用之处是可以让 Spring 的开发版本与 Maven Central 中部署的其他库一起使用; 这个仓库还包含一个捆绑包分发 zip 文件, 其中包含捆绑在一起的所有 Spring jar, 以便于下载  

所以你需要决定的第一件事是如何管理你的依赖: 我们一般推荐使用类似 Maven , Gradle 或者 Ivy 的自动化系统, 但是你也可以手工的通过下载所有的 jars 来完成  
以下是 Spring 构件的列表; 对于每个模块更多完整的描述见 [2.2 章节, 框架模块](https://docs.spring.io/spring/docs/4.3.20.RELEASE/spring-framework-reference/html/overview.html#overview-modules)

| GroupId | ArtifactId | Description |
| :--- | :--- |:--- |
| org.springframework | spring-aop | 基于代理的 AOP 支持 |
| org.springframework | spring-aspects | 基于 AspectJ 的切面 |
| org.springframework | spring-beans | Beans 支持, 包括 Groovy |
| org.springframework | spring-context | 应用程序运行时上下文, 包括调度和远程抽象 |
| org.springframework | spring-context-support | 支持将常见的第三方库集成到 Spring 应用程序上下文中的类 |
| org.springframework | spring-core | 用于其他 Spring 模块的核心工具类 |
| org.springframework | spring-expression | Spring 表达式语言 (SpEL) |
| org.springframework | spring-instrument | 用于 JVM 引导的工具代理程序 |
| org.springframework | spring-instrument-tomcat | 用于 Tomcat 的工具代理程序 |
| org.springframework | spring-jdbc | JDBC 支持包, 包括 DataSource 启动和 JDBC 访问支持 |
| org.springframework | spring-jms | JMS 支持包, 包括发送/接收 JMS 消息的帮助类 |
| org.springframework |spring-messaging | 支持消息传递体系结构和协议 |
| org.springframework | spring-orm | 对象/关系映射, 包括 JPA 和 Hibernate 的支持 |
| org.springframework | spring-oxm | 对象/XML 映射 |
| org.springframework | spring-test | 支持单元测试和集成测试 Spring 组件 |
| org.springframework | spring-tx | 事务基础架构, 包括 DAO 支持和 JCA 整合|
| org.springframework | spring-web | 基础 Web 支持, 包括 Web 客户端和基于 Web 的远程处理 |
| org.springframework | spring-webmvc | 用于 Servlet 栈的基于 HTTP 的模型-视图-控制器和 REST 终端 |
| org.springframework | spring-webmvc-portlet | 用于 Portlet 环境的 MVC 实现 |
| org.springframework | spring-websocket | WebSocket 和 SockJS 基础架构, 包括 STOMP 消息支持|

###### Spring 依赖和依赖 Spring
尽管 Spring 对大量的企业和其他外部工具提供了整合和支持, 但它刻意将自己的强依赖性保持绝对最小化: 在某些场景为了简单的使用 Spring 你不应该定位和下载 (即使是自动的) 大量的 jar 库; 对于基础的依赖注入仅需要一个强制的外部依赖, 那就是日志记录 (更多的细节叙述见以下日志记录选项)  
接下来我们概述了配置依赖于 Spring 的应用程序所需的基本步骤, 第一是使用 Maven 或者 Gradle 或者 Lvy; 在所有的示例中, 如果有任何的不清楚, 参考你的依赖管理系统的文档, 或者查看一些示例代码 - 当构建时 Spring 本身使用 Gradle 管理依赖, 并且我们的示例大多是使用 Gradle 或者 Maven

###### Maven 依赖管理
如果你使用 [Maven](https://maven.apache.org/) 管理依赖, 你甚至不需要提供明确的日志记录依赖; 例如, 创建一个应用程序上下文和使用依赖注入配置一个应用程序, 你的 Maven 依赖将如下所示:
```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.3.20.RELEASE</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```
注意范围可以声明为运行时, 如果你不需要使用 Spring APIs 进行编译, 这通常是基本依赖注入用例的场景  
以上示例是使用 Maven 中央仓库; 为了使用 Spring Maven 仓库 (例如: 为了里程碑版或者开发快照版), 你需要在你的 Maven 配置中指定仓库位置; 对于发行版:
```
<repositories>
    <repository>
        <id>io.spring.repo.maven.release</id>
        <url>http://repo.spring.io/release/</url>
        <snapshots><enabled>false</enabled></snapshots>
    </repository>
</repositories>
```
对于里程碑版
```
<repositories>  
    <repository>
        <id>io.spring.repo.maven.milestone</id>
        <url>http://repo.spring.io/milestone/</url>
        <snapshots><enabled>false</enabled></snapshots>
    </repository>
</repositories>
```
对于快照版
```
<repositories>
    <repository>
        <id>io.spring.repo.maven.snapshot</id>
        <url>http://repo.spring.io/snapshot/</url>
        <snapshots><enabled>true</enabled></snapshots>
    </repository>
</repositories>
```

###### Maven "Bill Of Materials" 依赖
使用 Maven 时, 可能会意外混合不同版本的 Spring JARs; 例如, 你可能会发现一个第三方库, 或者另一个 Spring 项目, 从传递性依赖中拉取到一个旧的发现版; 如果你自己忘记明确的声明一个直接依赖, 可能出现各种意想不到的问题  
为了解决这个问题, Maven 支持了 "bill of materials" (BOM) 依赖的概念; 你可以导入 `springframework-bom` 到你的 `dependencyManagement ` 中以确保所有的 Spring 依赖 (无论是直接或者传递性依赖) 是相同的版本
```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-framework-bom</artifactId>
            <version>4.3.20.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
使用 BOM 的另一个好处是当依赖于 Spring 框架的构件时你不在需要指定 `<version>` 属性
```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
<dependencies>
```

###### Gradle 依赖管理
为了与 Gradle 构建系统一起使用 Spring 仓库, 需要在 `repositories` 中包含正确的 URL:
```
repositories {
    mavenCentral()
    // and optionally...
    maven { url "http://repo.spring.io/release" }
}
```
你可以根据需要将 `repositories` URL 修改为 /release, /milestone 或者 /snapshot; 一旦一个仓库被配置, 你可以使用 Gradle 的方式声明依赖
```
dependencies {
    compile("org.springframework:spring-context:4.3.20.RELEASE")
    testCompile("org.springframework:spring-test:4.3.20.RELEASE")
}
```

###### Ivy 依赖管理
如果你更喜欢使用 [Ivy](https://ant.apache.org/ivy) 来管理依赖, 这里有类似的配置选项  
为了配置 Ivy 指向 Spring 仓库, 添加以下解析器到你的 `ivysettings.xml` 文件中:
```
<resolvers>
    <ibiblio name="io.spring.repo.maven.release"
            m2compatible="true"
            root="http://repo.spring.io/release/"/>
</resolvers>
```
你可以根据需要将 `root` URL 修改为 /release, /milestone 或者 /snapshot  
59/5000
配置完成后, 你可以按常规方式添加依赖; 例如 (在 ivy.xml):
```
<dependency org="org.springframework"
    name="spring-core" rev="4.3.20.RELEASE" conf="compile->runtime"/>
```

###### 发行版 Zip 文件
虽然使用构建系统可以支持依赖管理, 也是获取 Spring 框架的推荐方式, 但仍可以下载一个发行版 zip 文件  
发行版 Zips 被发布在 Spring Maven 仓库中 (这只是为了方便起见, 你不需要 Maven 或任何其他构建系统来下载它们)  
要下载分发 zip, 打开一个 web 浏览器到 ` http://repo.spring.io/release/org/springframework/spring`, 并且选择你想要的合适的版本子目录; 发行版文件以 `-dist-zip` 结尾, 例如 spring-framework-{spring-version}-RELEASE-dist.zip; 对于 [milestones](https://repo.spring.io/milestone/org/springframework/spring) 和 [snapshots](https://repo.spring.io/snapshot/org/springframework/spring) 也有公开发行版

##### 日志记录
对于 Spring 日志记录是非常重要的依赖; a) 它是唯一的强制性外部依赖, b) 每个人都想看见他们使用的工具的一些输出, c) Spring 集成了许多其他工具, 所有这些工具都可以选择日志记录依赖; 应用程序开发者的目标之一是对于整个应用程序在一个集中位置统一日志记录配置, 包括所有的外部组件; 这可能比以往更难, 因为现在有如此多的日志记录框架选择  
在 Spring 中强制的日志记录依赖
