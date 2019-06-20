### ApplicationContext 的其他功能
正如章节介绍中所讨论的, `org.springframework.beans.factory` 包提供了管理和操作 bean 的基本功能, 包括以编程方式;  `org.springframework.context` 包添加了 `ApplicationContext` 接口, 该接口扩展了 `BeanFactory` 接口, 此外还扩展了其他接口, 以更面向应用程序框架的方式提供其他功能; 许多人以完全声明的方式使用 `ApplicationContext`, 甚至不以编程方式创建它, 而是依赖于诸如 `ContextLoader` 之类的支持类来自动实例化 `ApplicationContext`, 作为 Java EE Web 应用程序的正常启动过程的一部分  
为了以更加面向框架的样式增强 `BeanFactory` 功能, 上下文包还提供以下功能
- 通过 `MessageSource` 接口访问 i18n 风格的消息
- 通过 `ResourceLoader` 接口访问 URL 和文件等资源
- 事件发布, 即通过使用 `ApplicationEventPublisher` 接口实现 `ApplicationListener` 接口的 bean
- 加载多个 (分层) 上下文, 允许每个上下文通过 `HierarchicalBeanFactory` 接口关注一个特定层, 例如应用程序的 Web 层

#### 使用 MessageSource 进行国际化
`ApplicationContext` 接口扩展了一个名为 `MessageSource` 的接口, 因此提供了国际化 (i18n) 功能; Spring 还提供了 `HierarchicalMessageSource` 接口, 它可以分层次地解析消息; 这些接口共同构成了 Spring 影响消息解析的基础; 这些接口上定义的方法包括
- `String getMessage(String code, Object[] args, String default, Locale loc)`: 用于从 `MessageSource` 检索消息的基本方法; 如果未找到指定区域设置的消息, 则使用默认消息; 传入的任何参数都使用标准库提供的 `MessageFormat` 功能成为替换值
- `String getMessage(String code, Object[] args, Locale loc)`: 与前一个方法基本相同, 但有一点不同: 不能指定默认消息; 如果找不到该消息, 则抛出 `NoSuchMessageException`
- `String getMessage(MessageSourceResolvable resolvable, Locale locale)`: 上述方法中使用的所有属性也包装在名为 `MessageSourceResolvable` 的类中, 你可以将此方法与此方法一起使用

加载 `ApplicationContext` 时, 它会自动搜索在上下文中定义的 `MessageSource` bean; bean 必须具有名称 `messageSource`; 如果找到这样的 bean, 则对前面方法的所有调用都被委托给消息源; 如果未找到任何消息源, `ApplicationContext` 将尝试查找包含具有相同名称的 bean 的父级; 如果是, 则将该 bean 用作 `MessageSource`; 如果 `ApplicationContext` 找不到任何消息源, 则会实例化一个空的 `DelegatingMessageSource`, 以便能够接受对上面定义的方法的调用  
Spring 提供了两个 `MessageSource` 实现, `ResourceBundleMessageSource` 和 `StaticMessageSource`; 两者都实现了 `HierarchicalMessageSource` 以便进行嵌套消息传递; `StaticMessageSource` 很少使用, 但提供了以编程方式向源添加消息; `ResourceBundleMessageSource` 显示在以下示例中
```
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```
在该示例中, 假设你在类路径中定义了三个资源包, 称为 `format, exceptions, windows` 任何解决消息的请求都将以 JDK 标准的方式处理, 通过 `ResourceBundles` 解析消息; 出于示例的目的, 假设上述两个资源包文件的内容是...
```
# in format.properties
message=Alligators rock!
```
```
# in exceptions.properties
argument.required=The {0} argument is required.
```
下一个示例中显示了执行 `MessageSource` 功能的程序; 请记住, 所有 `ApplicationContext` 实现也都是 `MessageSource` 实现, 因此可以强制转换为 `MessageSource` 接口
```
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", null);
    System.out.println(message);
}
```
上述程序产生的结果将是
```
Alligators rock!
```
总而言之, `MessageSource` 是在名为 `beans.xml` 的文件中定义的, 该文件存在于类路径的根目录中; `messageSource` bean 定义通过其 `basenames` 属性引用许多资源包; 在列表中传递给 `basenames` 属性的三个文件作为类路径根目录下的文件存在, 分别称为 `format.properties, exceptions.properties, windows.properties`  
下一个示例显示传递给消息查找的参数; 这些参数将转换为字符串并插入到查找消息中的占位符中
```
<beans>

    <!-- this MessageSource is being used in a web application -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- lets inject the above MessageSource into this POJO -->
    <bean id="example" class="com.foo.Example">
        <property name="messages" ref="messageSource"/>
    </bean>

</beans>
```
```
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", null);
        System.out.println(message);
    }
}
```
调用 `execute()` 方法得到的结果将是...
```
The userDao argument is required.
```
关于国际化 (i18n), Spring 的各种 `MessageSource` 实现遵循与标准 JDK `ResourceBundle` 相同的区域设置解析和回退规则; 简而言之, 继续前面定义的示例 `messageSource`, 如果要根据 `British(en-GB)` 语言环境解析消息, 则应分别创建名为 `format_en_GB.properties, exceptions_en_GB.properties, windows_en_GB.properties` 的文件  
通常, 区域设置解析由应用程序的周围环境管理; 在此示例中, 将手动指定将解析 (英国) 消息的区域设置
```
# in exceptions_en_GB.properties
argument.required=Ebagum lad, the {0} argument is required, I say, required.
```
```
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```
上述程序产生的结果将是
```
Ebagum lad, the 'userDao' argument is required, I say, required.
```
你还可以使用 `MessageSourceAware` 接口获取对已定义的任何 `MessageSource` 的引用; 在创建和配置 bean 时, 应用程序上下文的 `MessageSource` 会注入实现 `MessageSourceAware` 接口的 `ApplicationContext` 中定义的任何 bean  
> 作为 `ResourceBundleMessageSource` 的替代, Spring 提供了一个 `ReloadableResourceBundleMessageSource` 类; 此变体支持相同的包文件格式, 但比基于标准 JDK 的 `ResourceBundleMessageSource` 实现更灵活; 特别是, 它允许从任何 Spring 资源位置 (不仅仅是从类路径) 读取文件, 并支持 `bundle` 属性文件的热重新加载 (同时有效地在它们之间缓存它们) 有关详细信息, 请查看 `ReloadableResourceBundleMessageSource` 的 javadocs

#### 标准的和自定义的事件
`ApplicationContext` 中的事件处理是通过 `ApplicationEvent` 类和 `ApplicationListener` 接口提供的; 如果将实现 `ApplicationListener` 接口的 bean 部署到上下文中, 则每次将 `ApplicationEvent` 发布到 `ApplicationContext` 时, 都会通知该 bean; 从本质上讲, 这是标准的 `Observer` 设计模式
>从 Spring 4.2 开始, 事件基础结构得到了显着改进, 并提供了基于注释的模型以及发布任意事件的能力, 这是一个不一定从 `ApplicationEvent` 扩展的对象; 当发布这样的对象时, 我们将它包装在一个事件中

Spring 提供了以下标准的事件 (内建事件)

| 事件 | 说明 |
| :--- | :--- |
| ContextRefreshedEvent | 在初始化或刷新 `ApplicationContext` 时发布; 例如, 使用 `ConfigurableApplicationContext` 接口上的 `refresh()` 方法; 这里的 "Initialized" 表示加载所有 bean, 检测并激活 post-processors bean, 预先实例化单例, 并准备好使用 `ApplicationContext` 对象; 只要上下文尚未关闭, 只要所选的 `ApplicationContext` 实际支持这种 "热" 刷新, 就可以多次触发刷新; 例如, `XmlWebApplicationContext` 支持热刷新, 但 `GenericApplicationContext` 不支持 |
| ContextStartedEvent | 在 `ApplicationContext` 启动时发布, 使用 `ConfigurableApplicationContext` 接口上的 `start()` 方法; 这里的 "已启动" 意味着所有生命周期 bean 都会收到明确的启动信号; 通常, 此信号用于在显式停止后重新启动 Bean, 但它也可用于启动尚未为自动启动配置的组件, 例如尚未在初始化时启动的组件 |
| ContextStoppedEvent | 使用 `ConfigurableApplicationContext` 接口上的 `stop()` 方法在 `ApplicationContext` 停止时发布; 这里的 "停止" 意味着所有生命周期 bean 都会收到明确的停止信号; 可以通过 `start()` 调用重新启动已停止的上下文 |
| ContextClosedEvent | 在 `ApplicationContext` 关闭时发布, 使用 `ConfigurableApplicationContext` 接口上的 `close()` 方法; 这里的 "封闭" 意味着所有单例 bean 都被销毁; 封闭的环境达到了生命的终点, 它无法刷新或重新启动 |
| RequestHandledEvent | 一个特定于 Web 的事件, 告诉所有 bean 已经为 HTTP 请求提供服务; 请求完成后发布此事件; 此事件仅适用于使用 Spring 的 `DispatcherServlet` 的 Web 应用程序 |

你还可以创建和发布自己的自定义事件; 这个例子演示了一个扩展 Spring 的 `ApplicationEvent` 基类的简单类
```
public class BlackListEvent extends ApplicationEvent {

    private final String address;
    private final String content;

    public BlackListEvent(Object source, String address, String content) {
        super(source);
        this.address = address;
        this.content = content;
    }

    // accessor and other methods...
}
```
要发布自定义 `ApplicationEvent`, 请在 `ApplicationEventPublisher` 上调用 `publishEvent()` 方法; 通常, 这是通过创建一个实现 `ApplicationEventPublisherAware` 并将其注册为 Spring bean 的类来完成的; 以下示例演示了这样一个类
```
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blackList;
    private ApplicationEventPublisher publisher;

    public void setBlackList(List<String> blackList) {
        this.blackList = blackList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String content) {
        if (blackList.contains(address)) {
            publisher.publishEvent(new BlackListEvent(this, address, content));
            return;
        }
        // send email...
    }
}
```
在配置时, Spring 容器将检测到 `EmailService` 实现 `ApplicationEventPublisherAware` 并将自动调用 `setApplicationEventPublisher()`; 实际上, 传入的参数将是 Spring 容器本身; 你只需通过其 `ApplicationEventPublisher` 接口与应用程序上下文进行交互  
要接收自定义 `ApplicationEvent`, 请创建一个实现 `ApplicationListener` 的类并将其注册为 Spring bean; 以下示例演示了这样一个类
```
public class BlackListNotifier implements ApplicationListener<BlackListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```
请注意, `ApplicationListener` 通常使用自定义事件的类型 `BlackListEvent` 进行参数化; 这意味着 `onApplicationEvent()` 方法可以保持类型安全, 从而避免任何向下转换的需要; 你可以根据需要注册任意数量的事件侦听器, 但请注意, 默认情况下, 事件侦听器会同步接收事件; 这意味着 `publishEvent()` 方法将阻塞, 直到所有侦听器都已完成对事件的处理; 这种同步和单线程方法的一个优点是, 当侦听器接收到事件时, 如果事务上下文可用, 它将在发布者的事务上下文内运行; 如果需要另一个事件发布策略, 请参阅 Spring的 `ApplicationEventMulticaster` 接口的 javadoc  
以下示例显示了用于注册和配置上述每个类的 bean 定义
```
<bean id="emailService" class="example.EmailService">
    <property name="blackList">
        <list>
            <value>known.spammer@example.org</value>
            <value>known.hacker@example.org</value>
            <value>john.doe@example.org</value>
        </list>
    </property>
</bean>

<bean id="blackListNotifier" class="example.BlackListNotifier">
    <property name="notificationAddress" value="blacklist@example.org"/>
</bean>
```
总而言之, 当调用 `emailService` bean 的 `sendEmail()` 方法时, 如果有任何电子邮件应该被列入黑名单, 则会发布 `BlackListEvent` 类型的自定义事件; `blackListNotifier` bean 注册为 `ApplicationListener`, 因此接收 `BlackListEvent`, 此时它可以通知相关方
>Spring 的事件机制是为在同一应用程序上下文中的 Spring bean 之间的简单通信而设计的; 但是, 对于更复杂的企业集成需求, 单独维护的 Spring Integration 项目为构建基于众所周知的 Spring 编程模型的轻量级, 面向模式, 事件驱动的体系结构提供了完整的支持

##### 基于注解的事件监听器
从 Spring 4.2 开始, 可以通过 `EventListener` 注解在托管 bean 的任何公共方法上注册事件侦听器;  `BlackListNotifier` 可以如下重写
```
public class BlackListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlackListEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```
如上所示, 方法签名再次声明它侦听的事件类型, 但这次使用灵活的名称并且没有实现特定的侦听器接口; 只要实际事件类型在其实现层次结构中解析通用参数, 也可以通过泛型缩小事件类型  
如果你的方法应该监听多个事件, 或者你想要根据任何参数进行定义, 那么也可以在注解本身上指定事件类型
```
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    ...
}
```
还可以通过注解的 `condition` 属性添加额外的运行时过滤, 该注释定义应该匹配的 `SpEL` 表达式, 以实际调用特定事件的方法  
例如, 如果事件的 `content` 属性等于 `foo`, 则可以重写我们的通知程序以仅调用
```
@EventListener(condition = "#blEvent.content == 'foo'")
public void processBlackListEvent(BlackListEvent blEvent) {
    // notify appropriate parties via notificationAddress...
}
```
每个 `SpEL` 表达式都针对专用上下文进行评估; 下一个表列出了可用于上下文的项目, 因此可以将它们用于条件事件处理

| 名称 | 位置 | 描述 | 示例 |
| :--- | :--- | :--- | :--- |
| Event | root object | 真实的 `ApplicationEvent` | `#root.event` |
| Arguments array | root object | 用于调用目标的参数 (作为数组) | `#root.args[0]` |
| Argument name | evaluation context | 任何方法参数的名称; 如果由于某种原因名称不可用 (例如没有调试信息), 参数名称也可以在 `#a<#arg>` 下获得, 其中 `#arg` 代表参数索引 (从0开始) | `#blEvent` 或 `#a0` (也可以使用 `#p0` 或 `#p<#arg>` 表示法作为别名) |

请注意, `#root.event` 允许你访问基础事件，即使你的方法签名实际上是指已发布的任意对象  
如果你需要发布一个事件作为处理另一个事件的结果, 只需更改方法签名以返回应该发布的事件, 例如
```
@EventListener
public ListUpdateEvent handleBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress and
    // then publish a ListUpdateEvent...
}
```
>[异步监听器](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#context-functionality-events-async) 不支持此功能

这个新方法将为上面方法处理的每个 `BlackListEvent` 发布一个新的 `ListUpdateEvent`; 如果你需要发布多个事件, 请返回一个事件集合

##### 异步监听器
如果你希望特定监听器异步处理事件, 只需重用 [常规](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/scheduling.html#scheduling-annotation-support-async) `@Async` [支持](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/scheduling.html#scheduling-annotation-support-async)
```
@EventListener
@Async
public void processBlackListEvent(BlackListEvent event) {
    // BlackListEvent is processed in a separate thread
}
```
使用异步事件时请注意以下限制
- 如果事件侦听器抛出异常, 它将不会传播给调用者, 请检查 `AsyncUncaughtExceptionHandler` 以获取更多详细信息
- 此类事件监听器无法发送回复; 如果你需要作为处理结果发送另一个事件, 请注入 `ApplicationEventPublisher` 以手动发送事件

##### 排序监听器
如果需要在另一个之前调用侦听器, 只需将 `@Order` 注解添加到方法声明中
```
@EventListener
@Order(42)
public void processBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```
##### 泛型事件
你还可以使用泛型来进一步定义事件的结构; 考虑一个 `EntityCreatedEvent<T>`, 其中 `T` 是创建的实际实体的类型; 你可以创建以下监听器定义以仅接收 `Person` 的 `EntityCreatedEvent`
```
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    ...
}
```
由于类型擦除, 这仅在被触发的事件解析事件侦听器在其上过滤的泛型参数 (类似于 `PersonCreatedEvent` 类扩展 `EntityCreatedEvent<Person>{...}`) 时才有效  
在某些情况下, 如果所有事件都遵循相同的结构 (这应该是上述事件的情况), 这可能会变得相当繁琐; 在这种情况下, 你可以实现 `ResolvableTypeProvider` 来指导框架超出运行时环境提供的范围
```
public class EntityCreatedEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {

    public EntityCreatedEvent(T entity) {
        super(entity);
    }

    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(),
                ResolvableType.forInstance(getSource()));
    }
}
```
>这不仅适用于 `ApplicationEvent`, 也适用于你作为事件发送的任意对象

#### 方便地访问低级资源
为了最佳地使用和理解应用程序上下文, 用户通常应该熟悉 Spring 的资源抽象, 如 [第 8 章 "Resource" 一章](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/resources.html) 所述  
应用程序上下文是 `ResourceLoader`, 可用于加载资源; `Resource` 本质上是 JDK 类 `java.net.URL` 的功能更丰富的版本; 实际上, `Resource` 的实现在适当的地方包装了 `java.net.URL` 的实例; 资源可以透明的方式从几乎任何位置获取低级资源, 包括从类路径, 文件系统位置, 任何可用标准 URL 描述的位置, 以及一些其他变体; 如果资源位置字符串是没有任何特殊前缀的简单路径, 那么这些资源来自特定且适合于实际应用程序上下文类型  
你可以配置部署到应用程序上下文中的 bean, 以实现特殊的回调接口 `ResourceLoaderAware`, 在初始化时自动回调, 应用程序上下文本身作为 `ResourceLoader` 传入; 你还可以公开 `Resource` 类型的属性, 以用于访问静态资源, 它们将像任何其他属性一样注入其中; 你可以将这些 `Resource` 属性指定为简单的 `String` 路径, 并依赖于上下文自动注册的特殊 JavaBean `PropertyEditor`, 以便在部署 Bean 时将这些文本字符串转换为实际的 `Resource` 对象  
提供给 `ApplicationContext` 构造函数的位置路径实际上是资源字符串, 并且以简单形式适当地处理特定上下文实现; `ClassPathXmlApplicationContext` 将简单的位置路径视为类路径位置; 你还可以使用具有特殊前缀的位置路径 (资源字符串) 来强制从类路径或 `URL` 加载定义, 而不管实际的上下文类型如何

#### 方便的 Web 应用程序的 ApplicationContext 实例化
你可以使用例如 `ContextLoader` 以声明方式创建 `ApplicationContext` 实例; 当然, 你也可以使用其中一个 `ApplicationContext` 实现以编程方式创建 `ApplicationContext`
实例  
你可以使用 `ContextLoaderListener` 注册 `ApplicationContext`, 如下所示
```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
监听器检查 `contextConfigLocation` 参数; 如果参数不存在, 则监听器将 `/WEB-INF/applicationContext.xml` 用作默认值; 当参数确实存在时, 监听器使用预定义的分隔符 (逗号, 分号和空格) 分隔 `String`, 并将值用作将搜索应用程序上下文的位置; 还支持 `Ant` 样式的路径模式; 名称以 "Context.xml" 结尾的所有文件的 `/WEB-INF/*Context.xml`, 驻留在 "WEB-INF" 目录中, `/ WEB-INF/**/*Context.xml` 表示所有 "Context.xml" 结尾文件, 这些文件位于 "WEB-INF" 的任何子目录中

#### 将 Spring ApplicationContext 部署为Java EE RAR 文件
TODO


>**参考:**
[Additional capabilities of the ApplicationContext](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#context-introduction)
