### 代理机制
Spring AOP 使用 JDK 动态代理或 CGLIB 为给定目标对象创建代理; (只要有选择, JDK动态代理就是首选)  
如果要代理的目标对象实现至少一个接口, 则将使用 JDK 动态代理, 目标类型实现的所有接口都将被代理; 如果目标对象未实现任何接口, 则将创建 CGLIB 代理  
如果要强制使用 CGLIB 代理 (例如, 代理为目标对象定义的每个方法, 而不仅仅是那些由其接口实现的方法), 你可以这样做; 但是, 有一些问题需要考虑
- `final` 方法无法通知, 因为它们无法被覆写
- 从 Spring 3.2 开始, 不再需要将 CGLIB 添加到项目类路径中, 因为 CGLIB 类在 org.springframework 下重新打包并直接包含在 spring-core JAR 中; 这意味着基于 CGLIB 的代理支持 "正常工作" 的方式与 JDK 动态代理是一样的
- 从 Spring 4.0 开始, 代理对象的构造函数将不再被调用两次, 因为 CGLIB 代理实例将通过 Objenesis 创建; 只有当你的 JVM 不允许构造函数绕过时, 你才会看到来自 Spring 的 AOP 支持的双重调用和相应的调试日志条目

要强制使用 CGLIB 代理, 请将 `<aop:config>` 元素的 `proxy-target-class` 属性的值设置为 `true`
```
<aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
</aop:config>
```
要在使用 `@AspectJ` autoproxy 支持时强制 CGLIB 代理, 请将 `<aop:aspectj-autoproxy>` 元素的 `proxy-target-class` 属性设置为 `true`
```
<aop:aspectj-autoproxy proxy-target-class="true"/>
```
>多个 `<aop:config/>` 部分在运行时折叠为单个统一的自动代理创建器, 它应用指定的任何 `<aop:config/>` 部分 (通常来自不同的 XML bean 定义文件) 的最强代理设置; 这也适用于 `<tx:annotation-driven/>` 和 `<aop:aspectj-autoproxy/>`元素  
要明确: 在 `<tx:annotation-driven/>` 上使用 `proxy-target-class ="true"`, `<aop:aspectj-autoproxy/>` 或 `<aop:config/>` 元素将强制使用 CGLIB 代理它们

#### 理解 AOP 代理
Spring AOP 是基于代理的; 在编写自己的切面或使用 Spring Framework 提供的任何基于 Spring AOP 的切面之前, 掌握最后一个语句实际意味着什么的语义是非常重要的  
首先考虑一个场景, 其中有一个普通的, 未代理的, 没有特别关注它的直接对象引用, 如下面的代码片段所示
```
public class SimplePojo implements Pojo {

    public void foo() {
        // this next method invocation is a direct call on the 'this' reference
        this.bar();
    }

    public void bar() {
        // some logic...
    }
}
```
如果在对象引用上调用方法, 则直接在该对象引用上调用该方法, 如下所示  
![image](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/images/aop-proxy-plain-pojo-call.png)  
```
public class Main {

    public static void main(String[] args) {

        Pojo pojo = new SimplePojo();

        // this is a direct method call on the 'pojo' reference
        pojo.foo();
    }
}
```
当客户端代码具有的引用是代理时, 事情会稍微改变; 请考虑以下图表和代码段  
![image](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/images/aop-proxy-call.png)  
```
public class Main {

    public static void main(String[] args) {

        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());

        Pojo pojo = (Pojo) factory.getProxy();

        // this is a method call on the proxy!
        pojo.foo();
    }
}
```
这里要理解的关键是 `Main` 类的 `main(..)` 中的客户端代码具有对代理的引用; 这意味着对该对象引用的方法调用将是代理上的调用, 因此代理将能够委托给与该特定方法调用相关的所有拦截器 (通知); 但是, 一旦调用最终到达目标对象, 在这种情况下, SimplePojo 引用它可能对其自身进行的任何方法调用, 例如 `this.bar()` 或 `this.foo()` 将被调用, 这个是引用而不是代理; 这具有重要意义, 这意味着自我调用不会导致与方法调用相关的通知有机会执行  
好的, 那么该怎么办呢? 最好的方法 (这里松散地使用最好的术语) 是重构代码, 以便不会发生自我调用; 当然, 这确实需要你做一些工作, 但这是最好的, 最少侵入性的方法; 接下来的方法是绝对可怕的, 我几乎要谨慎地指出它, 因为它是如此可怕; 你可以 (惊讶) 通过这样做完全将你的类中的逻辑与 Spring AOP 联系起来
```
public class SimplePojo implements Pojo {

    public void foo() {
        // this works, but... gah!
        ((Pojo) AopContext.currentProxy()).bar();
    }

    public void bar() {
        // some logic...
    }
}
```
这完全将你的代码与 Spring AOP 耦合了起来, 它使类本身意识到它正在 AOP 上下文中使用, 它在AOP面前运行; 在创建代理时, 还需要一些额外的配置
```
public class Main {

    public static void main(String[] args) {

        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.adddInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());
        factory.setExposeProxy(true);

        Pojo pojo = (Pojo) factory.getProxy();

        // this is a method call on the proxy!
        pojo.foo();
    }
}
```
最后, 必须注意的是 AspectJ 没有这种自调用问题, 因为它不是基于代理的 AOP 框架

>**参考:**
[Proxying mechanisms](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-proxying)
