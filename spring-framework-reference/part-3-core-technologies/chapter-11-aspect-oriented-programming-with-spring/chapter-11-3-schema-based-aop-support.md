### 基于空间的 AOP 支持
如果你更喜欢基于 XML 的格式, 那么 Spring 还支持使用新的 "aop" 命名空间标记定义切面; 使用 `@AspectJ` 样式时支持完全相同的切入点表达式和通知类型, 因此在本节中我们将重点介绍新语法, 并将读者引用到上一节 ([第 11.2 节 "@AspectJ 支持"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj)) 中的讨论; 了解编写切入点表达式和通知参数的绑定  
要使用本节中描述的 aop 命名空间标记, 你需要导入 `spring-aop` 模式, 如 [第 41 章基于 XML 模式的配置](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/xsd-configuration.html) 中所述; 有关如何在 aop 空间中导入标记, 请参见 [第 41.2.7 节 "aop模式"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/xsd-configuration.html)  
在 Spring 配置中, 所有 `aspect` 和 `advisor` 元素必须放在 `<aop:config>` 元素中 (在应用程序上下文配置中可以有多个 `<aop:config>` 元素); `<aop:config>` 元素可以包含切入点, 通知和切面元素 (请注意, 这些元素必须按此顺序声明)  
>`<aop:config>` 样式的配置大量使用了 Spring 的自动代理机制; 如果你已经通过使用 `BeanNameAutoProxyCreator` 等使用显式自动代理, 这可能会导致问题 (例如通知不被编织); 通知的使用模式是仅使用 `<aop:config>` 样式, 或仅使用 `AutoProxyCreator` 样式

#### 声明一个切面
使用模式支持, 切面只是在 Spring 应用程序上下文中定义为 bean 的常规 Java 对象; 状态和行为在对象的字段和方法中捕获, 切入点和通知信息在 XML 中捕获  
使用 `<aop:aspect>` 元素声明切面, 并使用 `ref` 属性引用辅助 bean
```
<aop:config>
    <aop:aspect id="myAspect" ref="aBean">
        ...
    </aop:aspect>
</aop:config>

<bean id="aBean" class="...">
    ...
</bean>
```
支持切面的 bean (在本例中为 "aBean") 当然可以配置并依赖注入, 就像任何其他 Spring bean 一样

#### 声明一个切入点
可以在 `<aop:config>` 元素内声明命名切入点, 从而使切入点定义能够跨多个切面和通知共享  
表示服务层中任何业务服务执行的切入点可以定义如下
```
<aop:config>

    <aop:pointcut id="businessService"
        expression="execution(* com.xyz.myapp.service.*.*(..))"/>

</aop:config>
```
请注意, 切入点表达式本身使用与 [第 11.2 节 "@AspectJ 支持"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj) 中所述相同的 AspectJ 切入点表达式语言; 如果使用基于模式的声明样式, 则可以引用切入点表达式中类型 (@Aspects) 中定义的命名切入点; 定义上述切入点的另一种方法是
```
<aop:config>

    <aop:pointcut id="businessService"
        expression="com.xyz.myapp.SystemArchitecture.businessService()"/>

</aop:config>
```
假设你有 `SystemArchitecture` 切面, 如 ["共享公共切入点定义"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-common-pointcuts) 一节中所述  
在切面内部声明切入点与声明顶级切入点非常相似
```
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..))"/>

        ...

    </aop:aspect>

</aop:config>
```
在 `@AspectJ` 切面的方式大致相同, 使用基于模式的定义样式声明的切入点可能会收集连接点上下文; 例如, 以下切入点将 'this' 对象收集为连接点上下文并将其传递给通知
```
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..)) &amp;&amp; this(service)"/>

        <aop:before pointcut-ref="businessService" method="monitor"/>

        ...

    </aop:aspect>

</aop:config>
```
必须通过包含匹配名称的参数来声明通知以接收收集的连接点上下文
```
public void monitor(Object service) {
    ...
}
```
当组合切入点子表达式时, `&&` 在XML文档中是不方便的, 因此可以使用关键字 `and, or, not` 来分别代替 `&&, ||, !`; 例如, 之前的切入点可以更好地写为
```
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service..(..)) and this(service)"/>

        <aop:before pointcut-ref="businessService" method="monitor"/>

        ...
    </aop:aspect>
</aop:config>
```
请注意, 以这种 XML ID 引用方式定义的切入点, 不能用作命名切入点来形成复合切入点; 因此, 基于模式的定义样式中的命名切入点支持比 `@AspectJ` 样式提供的更有限

#### 声明通知
对于 `@AspectJ` 样式, 支持相同的五种通知类型, 它们具有完全相同的语义
```

```
##### 前置通知
在匹配的方法执行之前运行通知之前; 它使用 `<aop:before>` 元素在 `<aop:aspect>` 中声明
```
<aop:aspect id="beforeExample" ref="aBean">

    <aop:before
        pointcut-ref="dataAccessOperation"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```
这里 `dataAccessOperation` 是在顶部 (`<aop：config>`) 级别定义的切入点的 id; 要改为定义切入点, 请使用切入点属性替换 `pointcut-ref` 属性
```
<aop:aspect id="beforeExample" ref="aBean">

    <aop:before
        pointcut="execution(* com.xyz.myapp.dao.*.*(..))"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```
正如我们在讨论 `@AspectJ` 样式时所提到的, 使用命名切入点可以显着提高代码的可读性  
`method` 属性标识提供通知正文的方法 (doAccessCheck); 必须为包含通知的 `aspect` 元素引用的 bean 定义此方法; 在执行数据访问操作 (由切入点表达式匹配的方法执行连接点) 之前, 将调用切面 bean 上的 "doAccessCheck" 方法
##### 后置返回通知
在匹配的方法执行正常完成后返回通知运行, 它在 `<aop:aspect>` 中以与 `advice` 之前相同的方式声明; 例如
```
<aop:aspect id="afterReturningExample" ref="aBean">

    <aop:after-returning
        pointcut-ref="dataAccessOperation"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```
就像在 `@AspectJ` 样式中一样, 可以在通知体内获得返回值; 使用 `returns` 属性指定应将返回值传递到的参数的名称
```
<aop:aspect id="afterReturningExample" ref="aBean">

    <aop:after-returning
        pointcut-ref="dataAccessOperation"
        returning="retVal"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```
`doAccessCheck` 方法必须声明一个名为 `retVal` 的参数; 此参数的类型以与 `@AfterReturning` 所述相同的方式约束匹配; 例如, 方法签名可以声明为
```
public void doAccessCheck(Object retVal) {...
```
##### 后置异常通知
抛出通知执行时, 匹配的方法执行通过抛出异常退出; 它使用 `after-throwing` 元素在 `<aop:aspect>` 中声明
```
<aop:aspect id="afterThrowingExample" ref="aBean">

    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        method="doRecoveryActions"/>

    ...

</aop:aspect>   
```
就像在 `@AspectJ` 样式中一样, 可以在通知体内获取抛出的异常; 使用 `throwing` 属性指定应将异常传递到的参数的名称
```
<aop:aspect id="afterThrowingExample" ref="aBean">

    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        throwing="dataAccessEx"
        method="doRecoveryActions"/>

    ...

</aop:aspect>
```
`doRecoveryActions` 方法必须声明一个名为 `dataAccessEx` 的参数; 此参数的类型以与 `@AfterThrowing` 相同的方式约束匹配; 例如, 方法签名可以声明为
```
public void doRecoveryActions(DataAccessException dataAccessEx) {...
```

##### 后置(最终)通知
在 (最终) 通知运行之后, 匹配的方法执行退出; 它使用 `after` 元素声明
```
<aop:aspect id="afterFinallyExample" ref="aBean">

    <aop:after
        pointcut-ref="dataAccessOperation"
        method="doReleaseLock"/>

    ...

</aop:aspect>
```

##### 环绕通知
最后一种通知是环绕通知; 周围的通知围绕匹配的方法执行运行; 它有机会在方法执行之前和之后完成工作. 并确定方法实际上何时, 如何, 甚至是否实际执行; 如果你需要以线程安全的方式 (例如, 启动和停止计时器) 在方法执行之前和之后共享状态, 则经常使用环绕通知; 始终使用符合你要求的最不强大的通知形式; 如果前置通知可以实现就不要使用环绕通知  
使用 `aop:around` 元素声明环绕通知; 通知方法的第一个参数必须是 `ProceedingJoinPoint` 类型; 在通知的主体内, 在 `ProceedingJoinPoint` 上调用 `proceed()` 会导致执行基础方法; `proceed` 方法也可以调用传递给 `Object[]` --- 数组中的值将在进行时用作方法执行的参数; 有关调用继续使用 `Object []` 的说明请参阅 [环绕通知](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj-around-advice) 一节
```
<aop:aspect id="aroundExample" ref="aBean">

    <aop:around
        pointcut-ref="businessService"
        method="doBasicProfiling"/>

    ...

</aop:aspect>
```
`doBasicProfiling` 通知的实现与 `@AspectJ` 示例中的完全相同 (当然是去掉注释的)
```
public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
    // start stopwatch
    Object retVal = pjp.proceed();
    // stop stopwatch
    return retVal;
}
```

##### 通知参数
基于模式的声明样式支持完全类型化的通知, 方法与 `@AspectJ` 支持描述的方式相同 --- 通过名称匹配切入点参数和通知方法参数; 有关详细信息, 请参阅 ["通知参数"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj-advice-params) 一节; 如果你希望显式指定通知方法的参数名称 (不依赖于前面描述的检测策略), 那么这是使用 `advice` 元素的 `arg-names` 属性完成的, 该属性与在通知注解中的 "argNames" 属性的处理方式相同; 如 ["确定参数名称"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj-advice-params-names) 一节中所述; 例如：
```
<aop:before
    pointcut="com.xyz.lib.Pointcuts.anyPublicMethod() and @annotation(auditable)"
    method="audit"
    arg-names="auditable"/>
```
`arg-names` 属性接受以逗号分隔的参数名称列表  
下面是一个基于 XSD 的方法的一个稍微复杂的例子, 它说明了与一些强类型参数一起使用的一些通知
```
package x.y.service;

public interface FooService {

    Foo getFoo(String fooName, int age);
}

public class DefaultFooService implements FooService {

    public Foo getFoo(String name, int age) {
        return new Foo(name, age);
    }
}
```
接下来是切面; 请注意, `profile(..)` 方法接受许多强类型参数, 第一个参数恰好是用于继续方法调用的连接点: 此参数的存在表示 `profile(..)` 用作环绕通知
```
package x.y;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;

public class SimpleProfiler {

    public Object profile(ProceedingJoinPoint call, String name, int age) throws Throwable {
        StopWatch clock = new StopWatch("Profiling for '" + name + "' and '" + age + "'");
        try {
            clock.start(call.toShortString());
            return call.proceed();
        } finally {
            clock.stop();
            System.out.println(clock.prettyPrint());
        }
    }
}
```
最后, 这是为特定连接点执行上述通知所需的 XML 配置:
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- this is the object that will be proxied by Spring's AOP infrastructure -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- this is the actual advice itself -->
    <bean id="profiler" class="x.y.SimpleProfiler"/>

    <aop:config>
        <aop:aspect ref="profiler">

            <aop:pointcut id="theExecutionOfSomeFooServiceMethod"
                expression="execution(* x.y.service.FooService.getFoo(String,int))
                and args(name, age)"/>

            <aop:around pointcut-ref="theExecutionOfSomeFooServiceMethod"
                method="profile"/>

        </aop:aspect>
    </aop:config>

</beans>
```
如果我们有以下驱动程序脚本, 我们将在标准输出上获得类似的输出
```
import org.springframework.beans.factory.BeanFactory;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import x.y.service.FooService;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        BeanFactory ctx = new ClassPathXmlApplicationContext("x/y/plain.xml");
        FooService foo = (FooService) ctx.getBean("fooService");
        foo.getFoo("Pengo", 12);
    }
}
```
```
StopWatch 'Profiling for 'Pengo' and '12'': running time (millis) = 0
-----------------------------------------
ms     %     Task name
-----------------------------------------
00000  ?  execution(getFoo)
```

##### 通知顺序
当多个通知需要在同一个连接点 (执行方法) 执行时, 排序规则如 ["通知排序"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj-advice-ordering) 一节中所述; 切面之间的优先级是通过将 `Order` 注解添加到支持切面的 bean 或通过让 bean 实现 `Ordered` 接口来确定的  

#### 引言
TODO

#### 切面实例化模型
模式定义方面唯一支持的实例化模型是单例模型, 未来的版本可能支持其他实例化模型

#### 通知器
TODO

#### 示例
TODO

>**参考:**  
[Schema-based AOP support](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-schema)
