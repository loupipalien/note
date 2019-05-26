### 自定义 bean 的性质

#### 生命周期回调
要与容器的 bean 生命周期管理进行交互, 可以实现 Spring  `InitializingBean` 和 `DisposableBean` 接口; 容器为前者调用 `afterPropertiesSet()`, 为后者调用 `destroy()` 以允许 bean 在初始化和销毁 bean 时执行某些操作
>JSR-250 `@PostConstruct` 和 `@PreDestroy` 注解通常被认为是在现代 Spring 应用程序中接收生命周期回调的最佳实践; 使用这些注解意味着你的 bean 不会耦合 Spring 特定的接口; 有关详细信息, 见 [第 7.9.8 小节 "@PostConstruct 和 @PreDestroy"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-postconstruct-and-predestroy-annotations)
>如果你不想使用 JSR-250 注解但仍希望删除耦合, 请考虑使用 `init-method` 和 `destroy-method` 对象定义元数据

在内部，Spring Framework 使用 `BeanPostProcessor` 实现来处理它可以找到的任何回调接口并调用适当的方法; 如果你需要自定义功能或其他生命周期行为, Spring 不提供开箱即用的功能, 你可以自己实现 `BeanPostProcessor`; 有关更多信息, 见 [第 7.8 小节 "容器扩展点"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-extension)  
除了初始化和销毁回调之外, Spring 管理的对象还可以实现 `Lifecycle` 接口, 以便这些对象可以参与由容器自身生命周期驱动的启动和关闭过程  
本节将描述了生命周期回调接口

##### 初始化回调
`org.springframework.beans.factory.InitializingBean` 接口允许 bean 在容器设置 bean 的所有必要属性后执行初始化工作; `InitializingBean` 接口指定一个方法:
```
void afterPropertiesSet() throws Exception;
```
建议你不要使用 `InitializingBean` 接口, 因为它会不必要地将代码耦合到 Spring; 或者, 使用 `@PostConstruct` 注解或指定 POJO 初始化方法; 对于基于 XML 的配置元数, 可以使用 `init-method` 属性指定具有 void 无参数签名的方法的名称; 使用 Java 配置, 你可以使用 `@Bean` 的 `initMethod` 属性, 见 ["接收生命周期回调"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-java-lifecycle-callbacks) 小节; 例如, 以下内容:
```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```
```
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```
与以下完全相同
```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```
```
public class AnotherExampleBean implements InitializingBean {

    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```
但不会将代码耦合到 Spring

##### 销毁回调
实现 `org.springframework.beans.factory.DisposableBean` 接口允许 bean 在包含它的容器被销毁时获得回调; `DisposableBean` 接口指定一个方法:
```
void destroy() throws Exception;
```
建议你不要使用 `DisposableBean` 回调接口, 因为它会不必要地将代码耦合到 Spring; 或者, 使用 `@PreDestroy` 注解或指定 bean 定义支持的泛型方法; 使用基于 XML 的配置元数据, 可以在 `<bean/>` 上使用 `destroy-method` 属性; 使用 Java 配置, 你可以使用 `@Bean` 的 `destroyMethod` 属性,  见 ["接收生命周期回调"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-java-lifecycle-callbacks) 小节; 例如, 以下内容:
```
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```
```
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```
与以下完全相同
```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```
```
public class AnotherExampleBean implements DisposableBean {

    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```
但不会将代码耦合到 Spring
>可以为 `<bean>` 元素的 `destroy-method` 属性分配一个特殊的 (推断的) 值, 该值指示 Spring 自动检测特定 `bean` 类 (任何实现 java.lang.AutoCloseable 或 java.io.Closeable 的类) 的 public `close` 或 `shutdown` 方法; 此特殊 (推断的) 值也可以在 `<beans>` 元素的 `default-destroy-method` 属性上设置, 以将此行为应用于整个 bean 集 (见 ["默认初始化和销毁方法" 小节](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-lifecycle-default-init-destroy-methods)); 请注意, 这是 Java 配置的默认行为

##### 默认的初始化和销毁方法
当你不使用特定于 Spring 的 `InitializingBean` 和 `DisposableBean` 回调接口编写初始化和销毁的方法回调时, 通常会编写名称为 `init(), initialize(), dispose()` 等方法; 理想情况下, 此类生命周期回调方法的名称在项目中是标准化的, 以便所有开发人员使用相同的方法名称并确保一致性  
你可以配置 Spring 容器以查找命名初始化并销毁每个 bean 上的回调方法名称; 这意味着, 作为应用程序开发人员, 你可以编写应用程序类并使用名为 `init()` 的初始化回调, 而无需为每个 bean 定义配置 `init-method="init"`属性; Spring IoC 容器在创建 bean 时调用该方法 (并且符合前面描述的标准生命周期回调协定), 此功能还强制使执行的初始化和销毁方法回调的命名约定一致  
假设你的初始化回调方法名为 `init()`, 而销毁回调方法名为 `destroy()`; 你的类将类似于以下示例中的类
```
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```
```
<beans default-init-method="init">

    <bean id="blogService" class="com.foo.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```
顶级 `<beans/>` 元素属性上存在 `default-init-method` 属性会导致 Spring IoC 容器将 bean 上的 `init` 方法识别为初始化方法回调; 当 bean 被创建和组装时, 如果 bean 类具有这样的方法, 则在适当的时候调用它
你可以通过在顶级 `<beans/>` 元素上使用 `default-destroy-method` 属性来类似地配置 `destroy` 方法回调 (在 XML 中)  
如果现有 bean 类已经具有与约定一致的变量命名的回调方法, 则可以通过使用 `<bean/>` 的 `init-method`和 `destroy-method` 属性指定 (在 XML 中) 方法名称来覆盖缺省值  
Spring 容器保证在为 bean 提供所有依赖项后立即调用已配置的初始化回调; 因此, 在原始 bean 引用上调用初始化回调, 这意味着 AOP 拦截器等尚未应用于 bean; 首先完全创建目标 bean, 然后应用具有其拦截器链的 AOP 代理 (例如) 如果目标 bean 和代理是分开定义的, 那么你的代码甚至可以绕过代理与原始目标 bean 交互; 因此, 将拦截器应用于 `init` 方法是不一致的, 因为这样做会将目标 bean 的生命周期与其代理/拦截器耦合在一起, 并在代码直接与原始目标 bean 交互时留下奇怪的语义

##### 联合生命周期机制
从 Spring 2.5 开始, 你有三个控制 bean 生命周期行为的选项: `InitializingBean` 和 `DisposableBean` 回调接口; 自定义 `init()` 和 `destroy()` 方法; 以及 `@PostConstruct` 和 `@PreDestroy` 注解; 你可以组合这些机制来控制给定的 bean
>如果为 bean 配置了多个生命周期机制, 并且每个机制配置了不同的方法名称, 则每个配置的方法都按下面列出的顺序执行; 但是, 如果为初始化方法配置了相同的方法名称 (例如, init() --- 对应多个这些执行一次该方法的生命周期机制, 如上一节中所述

为同一个 bean 配置的多个具有不同的初始化方法生命周期机制, 调用顺序如下:
- 注解 `@PostConstruct` 的方法
- `InitializingBean` 接口中定义的 `afterPropertiesSet()` 方法
- 自定义的 `init()` 方法

销毁方法也按相同的顺序调用
- 注解 `@PreDestroy` 的方法
- `DisposableBean` 接口中定义的 `destroy()` 方法
- 自定义的 `destroy()` 方法

##### 启动和关闭回调
Lifecycle 接口对具有自己生命周期要求的任意对象定义基本方法 (例如, 启动和停止某些后台进程):
```
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```
任何 Spring 管理的对象都可以实现该接口; 然后, 当 `ApplicationContext` 本身接收到开始和停止信号时, 例如, 对于运行时的停止/重新启动场景, 它会将这些调用级联到该上下文中定义的所有生命周期实现; 它通过委托给 `LifecycleProcessor` 来实现:
```
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```
请注意, `LifecycleProcessor` 本身是 `Lifecycle` 接口的扩展; 它还添加了另外两种方法来响应刷新和关闭的上下文
>请注意, 常规 `org.springframework.context.Lifecycle` 接口只是显式启动/停止通知的简单合约, 并不意味着在上下文刷新时自动启动; 考虑实现 `org.springframework.context.SmartLifecycle`, 以便对特定 bean 的自动启动进行细粒度控制 (包括启动阶段); 此外, 请注意在销毁之前不能保证停止通知: 在常规关闭时, 所有 `Lifecycle` bean 将在传播一般销毁回调之前首先收到停止通知; 但是, 在上下文生命周期中的热刷新或中止刷新尝试时, 只会调用 `destroy` 方法

启动和关闭调用的顺序非常重要; 如果任何两个对象之间存在 "依赖" 关系, 则依赖方将在其依赖之后启动, 并且它将在其依赖之前停止; 但是, 有时直接依赖性是未知的; 你可能只知道某种类型的对象应该在另一种类型的对象之前开始; 在这些情况下, `SmartLifecycle` 接口定义了另一个选项, 即在其超级接口 `Phased` 上定义的 `getPhase()` 方法
```
public interface Phased {

    int getPhase();
}
```
```
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

>**参考:**
[Customizing the nature of a bean](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-factory-nature)
