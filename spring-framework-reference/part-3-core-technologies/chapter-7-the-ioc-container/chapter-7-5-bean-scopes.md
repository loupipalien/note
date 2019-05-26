### Bean 范围
创建 bean 定义时, 可以创建用于创建由该 bean 定义的类的实际实例的配方; bean 定义是一个配方的想法很重要, 因为它意味着, 就像一个类一样, 你可以从一个配方创建许多对象实例  
你不仅可以控制要插入到从特定 bean 定义创建的对象的各种依赖项和配置值, 还可以控制从特定 bean 定义创建的对象的范围; 这种方法功能强大且灵活, 你可以选择通过配置创建的对象的范围, 而不必在 Java 类级别设置对象的范围l; 可以将 Bean 定义部署在多个范围之一中: 开箱即用的, Spring Framework 支持七个范围, 其中五个范围仅在你使用 Web 感知 `ApplicationContext` 时可用  
开箱即用支持以下范围, 你还可以创建一个 [自定义范围](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-custom)

| 范围 | 描述 |
| :--- | :--- |
| [singleton](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-singleton) | (默认的) 将每个 Spring IoC 容器的单个 bean 定义范围限定为单个对象实例 |
| [prototype](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-prototype) | 将单个 bean 定义范围限定为任意数量的对象实例 |
| [request](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-request) | 将单个 bean 定义范围限定为单个 HTTP 请求的生命周期; 也就是说, 每个 HTTP 请求都有自己的 bean 实例, 它是在单个 bean 定义的后面创建的; 仅在 Web 感知 Spring ApplicationContext 的上下文中有效 |
| [session](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-session) | 将单个 bean 定义范围限定为 HTTP 会话的生命周期; 仅在 Web 感知 Spring ApplicationContext 的上下文中有效 |
| [globalSession](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-global-session) | 将单个 bean 定义范围限定为全局 HTTP 会话的生命周期; 通常仅在 Portlet 上下文中使用时有效; 仅在 Web 感知 Spring ApplicationContext 的上下文中有效 |
| [application](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-application) | 将单个 bean 定义范围限定为 ServletContext 的生命周期; 仅在 Web 感知 Spring ApplicationContext 的上下文中有效 |
| [websocket](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/websocket.html#websocket-stomp-websocket-scope) | 将单个 bean 定义范围限定为 WebSocket 的生命周期; 仅在 Web 感知 Spring ApplicationContext 的上下文中有效 |
>从 Spring 3.0 开始, 线程范围可用, 但默认情况下未注册; 有关更多信息, 请参阅 `SimpleThreadScope` 的文档; 有关如何注册此范围或任何其他自定义范围的说明, 见 ["使用自定义范围" 小节](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-custom-using)

#### 单例范围
只管理单个 bean 的一个共享实例, 并且对具有与该 bean 定义匹配的 id 或 id 的 bean 的所有请求都会使 Spring 容器返回一个特定的 bean 实例  
换句话说, 当你定义 bean 定义并将其作为单例范围时, Spring IoC 容器只创建该 bean 定义所定义对象的一个实例; 此单例实例存储在此类单例 bean 的缓存中, 并且该命名 Bean 的所有后续请求和引用都将返回缓存对象  
![image](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/images/singleton.png)  
Spring 的单例 bean 概念不同于 Gang of Four (GoF) 模式书中定义的 Singleton 模式; GoF Singleton 对对象的范围进行硬编码, 使得每个 ClassLoader 创建一个且只有一个特定类的实例; Spring 单例的范围最好按容器和每个 bean 定义; 这意味着如果在单个 Spring 容器中为特定类定义一个 bean, 则 Spring 容器将创建该 bean 定义所定义的类的一个且仅一个实例; 单例范围是 Spring 中的默认范围; 例如, 要将 bean 定义为 XML 中的单例, 你可以编写
```
<bean id="accountService" class="com.foo.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.foo.DefaultAccountService" scope="singleton"/>
```

#### 原型范围
bean 的非单例原型范围部署导致每次发出对该特定 bean 的请求时都会创建一个新的 bean 实例; 也就是说, bean 被注入另一个 bean, 或者通过对容器的 getBean() 方法调用来请求它; 通常, 对所有有状态 bean 使用原型范围, 对无状态 bean使用单例范围  
下图说明了 Spring 原型范围; 数据访问对象 (DAO) 通常不配置为原型, 因为典型的 DAO 不保持任何会话状态; 这样更容易被图的核心的单例所重用  
![image](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/images/prototype.png)  
以下示例在 XML 中定义了一个原型 bean
```
<bean id="accountService" class="com.foo.DefaultAccountService" scope="prototype"/>
```
与其他作用域相比, Spring 不管理原型 bean 的完整生命周期: 容器实例化, 配置和组装原型对象, 并将其交给客户端, 而不再记录该原型实例; 因此, 尽管无论范围如何都在所有对象上调用初始化生命周期回调方法, 但在原型的情况下, 不会调用已配置的销毁生命周期回调; 客户端代码必须清理原型范围的对象并释放原型 bean 所持有的昂贵资源; 要使 Spring 容器释放原型范围的 bean 所拥有的资源, 请尝试使用自定义 [bean post-procerssor](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-extension-bpp), 它包含对需要清理的 bean 的引用  
在某些方面, Spring 容器关于原型范围 bean 的角色是 Java new 运算符的替代品; 超过该点的所有生命周期管理必须由客户端处理 (有关 Spring 容器中 bean 的生命周期的详细信息, 见 [第 7.6.1 节 "生命周期回调"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-lifecycle))

#### 依赖原型 bean 的单例 bean
当你使用具有依赖于原型 bean 的单例 bean 时, 请注意在实例化时解析依赖项; 因为如果要将依赖项将原型 bean 注入到单例 bean 中, 则会实例化一个新的原型 bean, 然后将依赖注入到单例 bean 中; 只有一个原型实例提供给单例 bean    
但是, 假设你希望单例 bean 在运行时重复获取原型 bean 的新实例; 你不能将原型 bean 依赖注入到你的单例 bean 中, 因为当 Spring 容器实例化单例 bean 并解析和注入其依赖项的过程只发生一次; 如果你需要在运行时多次使用原型 bean 的新实例, 见 [第 7.4.6 节 "方法注入"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-method-injection)

#### request, session, globalSession, application, websocket 范围
`request`, `session`, `globalSession`, `application`, `websocket` 范围仅在你使用 Web 感知的 Spring ApplicationContext 实现 (例如 XmlWebApplicationContext) 时才可用; 如果将这些范围与常规的 Spring IoC 容器 (如 ClassPathXmlApplicationContext) 一起使用, 则会抛出 `IllegalStateException`, 报错未知的 bean 范围
##### 初始化 web 配置
要在 `request`, `session`, `globalSession`, `application`, `websocket` 级别 (Web 范围的 bean) 支持 bean 的范围, 在定义 bean 之前需要一些的初始配置 (标准范围不需要这种初始设置, 单例和原型)  
如何完成此初始设置取决于你的特定 Servlet 环境  
如果你在 Spring Web MVC 中访问 scoped bean, 实际上是在 Spring `DispatcherServlet` 或 `DispatcherPortlet` 处理的请求中, 则不需要特殊设置: `DispatcherServlet` 和 `DispatcherPortlet` 已经公开了所有相关状态  
如果你使用 Servlet 2.5 Web 容器, 并且在 Spring 的 `DispatcherServlet` 之外处理请求 (例如, 使用 JSF  或 Struts 时), 则需要注册 `org.springframework.web.context.request.RequestContextListener` `ServletRequestListener`; 对于 Servlet 3.0+, 可以通过 `WebApplicationInitializer` 接口以编程方式完成; 或者对于旧容器, 将以下声明添加到 Web 应用程序的 web.xml 文件中
```
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```
或者, 如果你的侦听器设置存在问题, 请考虑使用 Spring 的 `RequestContextFilter`; 过滤器映射取决于周围的 Web 应用程序配置, 因此你必须根据需要进行更改
```
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```
`DispatcherServlet, RequestContextListener, RequestContextFilter` 都完全相同, 即将 HTTP 请求对象绑定到为该请求提供服务的 Thread; 这使得请求和会话范围的 bean 可以在调用链的下游进一步使用
##### request 范围
请考虑 bean 定义的以下 XML 配置
```
<bean id="loginAction" class="com.foo.LoginAction" scope="request"/>
```
Spring 容器通过对每个 HTTP 请求使用 loginAction bean 定义来创建 LoginAction bean 的新实例; 也就是说, loginAction bean 的范围是 HTTP 请求级别; 你可以根据需要更改创建的实例的内部状态, 因为从同一个 loginAction bean 定义创建的其他实例将不会在状态中看到这些更改; 它们特别针对单独要求; 当请求完成处理时, 将放弃作用于请求的 bean  
使用注释驱动的组件或 Java Config 时, 可以使用 `@RequestScope` 注释将组件分配给请求范围
```
@RequestScope
@Component
public class LoginAction {
    // ...
}
```
##### session 范围
请考虑 bean 定义的以下 XML 配置
```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
```
Spring 容器通过在单个 HTTP 会话的生存期内使用 userPreferences bean 定义来创建 UserPreferences bean 的新实例; 换句话说, userPreferences bean 在 HTTP 会话级别有效地作用域; 与请求范围的 bean 一样, 你可以根据需要更改创建的实例的内部状态, 因为知道同样使用从同一 userPreferences bean 定义创建的实例的其他 HTTP Session 实例不会在状态中看到这些更改, 因为它们特定于单个 HTTP 会话; 最终丢弃 HTTP 会话时, 也会丢弃作用于该特定 HTTP 会话的 bean  
使用注释驱动的组件或 Java Config 时, 可以使用 `@SessionScope` 注释将组件分配给会话范围
```
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```
##### global session 范围
请考虑 bean 定义的以下 XML 配置
```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="globalSession"/>
```
globalSession 范围类似于标准 HTTP 会话范围 (如上所述), 并且仅适用于基于 portlet 的 Web 应用程序的上下文; portlet 规范定义了构成单个 portlet Web 应用程序的所有 portlet 之间共享的全局会话的概念; 在 globalSession 作用域中定义的 Bean 的会作用 (或绑定) 到全局 portlet 会话的生存期  
如果编写基于 Servlet 的标准 Web 应用程序并将一个或多个 bean 定义为具有 globalSession 作用域, 同时使用标准 HTTP 会话作用域, 并且不会引发错误
##### application 范围
请考虑 bean 定义的以下 XML 配置
```
<bean id="appPreferences" class="com.foo.AppPreferences" scope="application"/>
```
Spring 容器通过对整个 Web 应用程序使用 appPreferences bean 定义一次来创建 AppPreferences bean 的新实例; 也就是说, appPreferences bean 的作用域是 ServletContext 级别, 存储为常规的 ServletContext 属性; 这有点类似于 Spring 单例 bean, 但在两个重要方面有所不同: 它是每个 ServletContext 的单例, 而不是每个 Spring的 `ApplicationContext` (在任何给定的 Web 应用程序中可能有几个), 它实际上是暴露的, 因此作为 ServletContext 属性可见
使用注释驱动的组件或 Java Config 时, 可以使用 `@ApplicationScope` 注释将组件分配给应用程序范围
```
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

##### scoped beans 作为依赖
Spring IoC 容器不仅管理对象 (bean) 的实例化, 还管理协作者 (或依赖关系) 的连接; 如果要将 (例如) HTTP 请求范围 bean 注入到具有较长寿命范围的另一个 bean 中, 你可以选择注入 AOP 代理来代替范围 bean; 也就是说, 你需要注入一个代理对象, 该对象公开与范围对象相同的公共接口, 但也可以从相关范围 (例如 HTTP 请求) 中检索真实目标对象, 并将方法调用委托给真实对象  
>你还可以在作为单例范围的 bean 之间使用 `<aop:scoped-proxy/>`, 然后引用通过可序列化的中间代理, 从而能够在反序列化时重新获取目标单例 bean
>当针对原型范围的 bean 声明 `<aop:scoped-proxy/>`时, 共享代理上的每个方法调用都将导致创建一个新的目标实例, 然后该调用将被转发到该目标实例
>此外, 范围代理不是以生命周期安全的方式访问较短范围 bean 的唯一方法, 你也可以简单地将你的注入点 (即构造函数/ setter 参数或自动装配字段) 声明为 `ObjectFactory<MyTargetBean>`, 允许 `getObject()` 调用在每次需要时按需检索当前实例 --- 而无需保留实例或单独存储它
>作为扩展变体, 你可以声明 `ObjectProvider<MyTargetBean>`, 它提供了几个额外的访问变体, 包括 `getIfAvailable` 和 `getIfUnique`
>JSR-330 的变量称为 Provider, 与 `Provider<MyTargetBean>`声明一起使用, 并且对每次检索尝试都使用相应的 `get()` 调用; 有关 JSR-330 整体的更多详细信息见 [这里](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-standard-annotations)

以下示例中的配置只有一行, 但了解 "为什么" 以及它背后的 "如何" 非常重要
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/>
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.foo.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```
为了创建这样的代理, 可以将子 `<aop:scoped-proxy/>` 元素插入到范围 bean 定义中 (请参阅 ["选择要创建的代理类型"小节](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes-other-injection-proxies) 和 [第 41 章基于 XML 模式的配置](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/xsd-configuration.html)); 为什么在 `request`, `session`, `globalSession` 和自定义范围级别定义 bean 的定义需要 `<aop:scoped-proxy/>` 元素? 让我们检查下面的单例 bean 定义, 并将其与你需要为上述范围定义的内容进行对比 (请注意, 以下 userPreferences bean 定义不完整)
```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```
在前面的示例中, 单例 bean userManager 注入了对 HTTP 会话范围的 bean userPreferences 的引用; 这里的重点是 userManager bean 是一个单例: 它将在每个容器中实例化一次, 并且它的依赖项 (在这种情况下只有一个, userPreferences bean) 也只注入一次; 这意味着 userManager bean 将仅对完全相同的 userPreferences 对象进行操作, 即最初注入的对象  
当将一个寿命较短的 scoped bean 注入一个寿命较长的 scoped bean 时, 这不是你想要的行为, 例如将一个 HTTP Session-scoped 合作 bean 作为依赖注入 singleton bean; 相反, 你需要一个 userManager 对象, 并且在 HTTP 会话的生命周期中, 你需要一个特定于所述 HTTP 会话的 userPreferences 对象; 因此, 容器创建一个对象, 该对象公开与 UserPreferences 类 (理想情况下是 UserPreferences 实例的对象) 完全相同的公共接口, 该对象可以从作用域机制 (HTTP请求, 会话等) 获取真实的UserPreferences 对象; 容器将此代理对象注入 userManager bean, 该 bean 不知道此 UserPreferences 引用是代理; 在此示例中, 当 UserManager 实例在依赖注入的 UserPreferences 对象上调用方法时, 它实际上是在代理上调用方法; 然后, 代理从 (在这种情况下) HTTP 会话中获取真实的 UserPreferences 对象, 并将方法调用委托给检索到的真实 UserPreferences 对象  
因此, 在将 `request-, session-, globalSession-scoped` bean 注入协作对象时, 你需要以下正确和完整的配置
```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```
##### 选择代理创建的类型
默认情况下, 当 Spring 容器为使用 `<aop:scoped-proxy/>` 元素标记的 bean 创建代理时, 将创建基于 CGLIB 的类代理
>CGLIB 代理只拦截公共方法调用! 不要在这样的代理上调用非公开方法; 它们不会被委托给实际的作用域目标对象

或者, 你可以通过为 `<aop:scoped-proxy/>` 元素的 `proxy-target-class` 属性的值指定 false, 将 Spring 容器配置为此类作用域 bean 创建基于 JDK 接口的标准代理; 使用基于 JDK 接口的代理意味着你不需要在应用程序类路径中使用其他库来实现此类代理; 但是, 这也意味着 scoped bean 的类必须至少实现一个接口, 并且注入 scoped bean 的所有协作者必须通过其中一个接口引用 bean
```
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.foo.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```
有关选择基于类或基于接口的代理的更多详细信息, 见 [第 11.6 节 "代理机制"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-proxying)

#### 自定义 scopes
TODO

>**参考:**  
[Bean scopes](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes)
