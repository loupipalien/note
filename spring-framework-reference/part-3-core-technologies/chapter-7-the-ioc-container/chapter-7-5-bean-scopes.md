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

>**参考:**  
[Bean scopes](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-scopes)
