### @AspectJ 支持
`@AspectJ` 指的是将切面声明为使用注解注释的常规 Java 类的样式; 作为 AspectJ 5 版本的一部分, AspectJ 项目引入了 `@AspectJ` 样式; Spring 使用 AspectJ 提供的库解释与 AspectJ 5 相同的注解, 用于切入点解析和匹配; AOP 运行时仍然是纯 Spring AOP, 并且不依赖于 AspectJ 编译器或 weaver  
>使用 AspectJ 编译器和 weaver 可以使用完整的 AspectJ 语言, 在 [第 11.8 节 "将 AspectJ 与 Spring 应用程序一起使用"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-using-aspectj) 中进行了讨论

#### 开启 @AspectJ 支持
要在 Spring 配置中使用 `@AspectJ` 切面, 你需要启用 Spring 支持, 以便基于 `@AspectJ` 方面配置 Spring AOP, 并根据这些切脉你是否通知来自动执行 bean; 我们的意思是, 如果 Spring 确定 bean 被一个或多个切面通知, 通过 autoproxying 它将自动生成该 bean 的代理以拦截方法调用并确保根据需要执行通知  
可以使用 XML 或 Java 样式配置启用 `@AspectJ` 支持; 在任何一种情况下, 你还需要确保 AspectJ 的 `aspectjweaver.jar` 库位于应用程序的类路径中 (版本 1.6.8 或更高版本); 该库可在 AspectJ 发行版的 "lib" 目录中或通过 Maven Central 存储库获得  
##### 使用 Java 配置开启 @AspectJ 支持
要使用 Java 配置 `@Configuration` 启用 `@AspectJ` 支持, 请添加 `@EnableAspectJAutoProxy` 注解
```
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```
##### 使用 XML 配置开启 @AspectJ 支持
要使用基于 XML 的配置启用 `@AspectJ` 支持, 请使用 `<aop:aspectj-autoproxy/>` 元素
```
<aop:aspectj-autoproxy/>
```
这假定你正在使用 [第 41 章基于 XML 模式的配置](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/xsd-configuration.html) 中所述的模式支持; 有关如何在 aop 命名空间中导入标记, 请参见 [第 41.2.7 节 "aop 模式"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/xsd-configuration.html#xsd-config-body-schemas-aop)

#### 声明一个切面
在启用 `@AspectJ` 支持的情况下, 在应用程序上下文中定义的任何 bean 都是一个 `@AspectJ` 切面的类 (具有 `@Aspect` 注解) 将由 Spring 自动检测并用于配置 Spring AOP; 以下示例显示了非常有用的方面所需的最小定义  
应用程序上下文中的常规 bean 定义, 指向具有 `@Aspect` 注解的 bean 类
```
<bean id="myAspect" class="org.xyz.NotVeryUsefulAspect">
    <!-- configure properties of aspect here as normal -->
</bean>
```
以及 `NotVeryUsefulAspect` 类定义, 用 `org.aspectj.lang.annotation.Aspect` 注解注释
```
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class NotVeryUsefulAspect {

}
```
切面 (使用 `@Aspect` 注解的类) 可能具有与任何其他类一样的方法和字段; 它们还可能包含切入点, 通知和引言 (类型间) 声明
>你可以在 Spring XML 配置中将切面类注册为常规 bean, 或者通过类路径扫描自动检测它们 --- 就像任何其他 Spring 管理的 bean 一样; 但是, 请注意 `@Aspect` 注解不足以在类路径中自动检测; 为此, 你需要添加单独的 `@Component` 注解 (或者根据 Spring 的组件扫描程序的规则添加符合条件的自定义构造型注释)
>在 Spring AOP 中, 不可能将切面本身作为其他切面的通知目标, 类上的 `@Aspect` 注解将其标记为切面, 因此将其从自动代理中排除

#### 声明一个切入点
回想一下, 切入点确定了感兴趣的连接点, 从而使我们能够控制通知何时执行; Spring AOP 仅支持 Spring bean 的方法执行连接点, 因此你可以将切入点视为匹配 Spring bean 上方法的执行; 切入点声明有两个部分: 一个包含名称和任何参数的签名, 以及一个精确确定我们感兴趣的方法执行的切入点表达式; 在 AOP 的 `@AspectJ` 注解样式中, 切入点签名由常规方法提供定义, 并使用 `@Pointcut` 注解指示切入点表达式 (用作切入点签名的方法必须具有 `void` 返回类型)  
以下示例将有助于区分切入点签名和切入点表达式; 以下示例定义名为 "anyOldTransfer" 的切入点, 该切入点将匹配名为 "transfer" 的任何方法的执行
```
@Pointcut("execution(* transfer(..))")// the pointcut expression
private void anyOldTransfer() {}// the pointcut signature
```
`@Pointcut` 注解值的切入点表达式形式是常规的 AspectJ 5 切入点表达式; 有关 AspectJ 的切入点语言的完整讨论, 请参阅 [AspectJ 编程指南](https://www.eclipse.org/aspectj/doc/released/progguide/index.html) (以及扩展, [AspectJ 5 开发人员手册](https://www.eclipse.org/aspectj/doc/released/adk15notebook/index.html)) 或 AspectJ 上的一本书, 例如 Colyer 的 "Eclipse AspectJ" 或 Ramnivas Laddad 的 "AspectJ in Action"

##### 支持的切入点指示符
Spring AOP 支持以下 AspectJ 切入点指示符 (PCD), 用于切入点表达式
```
其他切入点类型
完整的 AspectJ 切入点语言支持 Spring 中不支持的其他切入点指示符; 它们是: `call, get, set, preinitialization, staticinitialization, initialization, handler, adviceexecution, withincode, cflow, cflowbelow, if, @this, @withincode`; 在 Spring AOP 解释的切入点表达式中使用这些切入点指示符将导致抛出 IllegalArgumentException  
Spring AOP 支持的切入点指示符集可以在将来的版本中进行扩展, 以支持更多的 AspectJ 切入点指示符
```
- execution: 对于匹配方法执行连接点, 这是在使用 Spring AOP 时将使用的主要切入点指示符
- within: 限制匹配以连接某些类型中的点 (仅使用 Spring AOP 时执行在匹配类型中声明的方法)
- this: 限制匹配到连接点 (使用 Spring AOP 时执行方法) 其中 bean 引用 (Spring AOP 代理) 是给定类型的实例
- target: 限制匹配连接点 (使用 Spring AOP 时执行方法), 其中目标对象 (被代理的应用程序对象) 是给定类型的实例
- args: 限制匹配到连接点 (使用 Spring AOP 时执行方法), 其中参数是给定类型的实例
- @target: 限制匹配到连接点 (使用 Spring AOP 时执行方法), 其中执行对象的类具有给定类型的注解
- @args: 限制匹配到连接点 (使用 Spring AOP 时执行方法), 其中传递的实际参数的运行时类型具有给定类型的注解
- @within: 限制匹配以连接具有给定注解的类型中的点 (使用 Spring AOP 时在具有给定注解的类型中声明的方法的执行)
- @annotation: 限制匹配到连接点的对象, 其中连接点的对象 (在 Spring AOP 中执行的方法) 具有给定的注释

由于 Spring AOP 仅限制与方法执行连接点的匹配, 因此上面对切入点指示符的讨论给出了比在 AspectJ 编程指南中找到的更窄的定义; 此外, AspectJ 本身具有基于类型的语义, 并且在执行连接点, `this` 和 `target` 都引用同一个对象 --- 执行该方法的对象; Spring AOP 是一个基于代理的系统, 它区分代理对象本身 (绑定到此) 和代理后面的目标对象 (绑定到目标)
>由于 Spring 的 AOP 框架基于代理的特性, 目标对象内的调用根据定义不会被截获; 对于 JDK 代理, 只能拦截代理上的公共接口方法调用; 使用 CGLIB, 代理上的公共和受保护方法调用将被拦截, 甚至包括必要的包可见方法; 但是, 通过代理进行的常见交互应始终通过公共签名进行设计
>请注意, 切入点定义通常与任何截获的方法匹配; 如果切入点严格意义上是公开的, 即使在通过代理进行潜在非公共交互的 CGLIB 代理方案中, 也需要相应地定义切入点。
>如果你的拦截需要包括目标类中的方法调用甚至构造函数, 请考虑使用 Spring 驱动的 [原生 AspectJ 编织](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-aj-ltw) 而不是 Spring 的基于代理的 AOP 框架; 这构成了具有不同特征的不同 AOP 使用模式, 因此在做出决定之前一定要先熟悉编织

Spring AOP 还支持另一个名为 `bean` 的 PCD; 此 PCD 允许你将连接点的匹配限制为特定的命名 Spring bean, 或限制为一组命名的 Spring bean (使用通配符时); `bean` PCD 具有以下形式
```
bean(idOrNameOfBean)
```
`idOrNameOfBean` 标记可以是任何 Spring bean 的名称: 提供了使用 `*` 字符的有限通配符支持, 因此如果为 Spring bean 建立一些命名约定, 则可以非常轻松地编写 bean PCD 表达式以将其选中; 与其他切入点指示符的情况一样, bean PCD 可以使用 `&&, ||, !`
>请注意, bean PCD 仅在 Spring AOP 中受支持 --- 而不是在原生 AspectJ 编织中; 它是 AspectJ 定义的标准 PCD的 Spring 特定扩展, 因此不适用于 `@Aspect` 模型中声明的切面
>bean PCD 在实例级别 (基于 Spring bean 名称概念) 而不是仅在类型级别 (这是基于编织的 AOP 限制)运行; 基于实例的切入点指示符是 Spring 基于代理的 AOP 框架的一种特殊功能, 它与 Spring bean 工厂紧密集成, 通过名称可以自然而直接地识别特定的 bean

##### 组合切入点表达式
可以使用 `&&, ||, !` 组合切入点表达式; 也可以通过名称引用切入点表达式; 以下示例显示了三个切入点表达式: `anyPublicOperation` (如果方法执行连接点表示任何公共方法的执行, 则匹配); `inTrading` (如果方法执行在交易模块中, 则匹配) 和 `tradingOperation` (如果方法执行代表交易模块中的任何公共方法, 则匹配)
```
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {}

@Pointcut("within(com.xyz.someapp.trading..*)")
private void inTrading() {}

@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {}
```
如上所示, 最佳实践是从较小的命名组件构建更复杂的切入点表达式; 当按名称引用切入点时, 将应用常规 Java 可见性规则 (你可以看到相同类型的私有切入点, 层次结构中受保护的切入点, 任何地方的公共切入点等等); 可见性不会影响切入点匹配

##### 共享通用切入点的定义
使用企业应用程序时, 你经常需要从几个方面引用应用程序的模块和特定的操作集; 我们建议定义一个 "SystemArchitecture" 的切面, 为此目的捕获常见的切入点表达式; 典型的这种切面看起来如下
```
package com.xyz.someapp;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class SystemArchitecture {

    /**
     * A join point is in the web layer if the method is defined
     * in a type in the com.xyz.someapp.web package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.web..*)")
    public void inWebLayer() {}

    /**
     * A join point is in the service layer if the method is defined
     * in a type in the com.xyz.someapp.service package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.service..*)")
    public void inServiceLayer() {}

    /**
     * A join point is in the data access layer if the method is defined
     * in a type in the com.xyz.someapp.dao package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.dao..*)")
    public void inDataAccessLayer() {}

    /**
     * A business service is the execution of any method defined on a service
     * interface. This definition assumes that interfaces are placed in the
     * "service" package, and that implementation types are in sub-packages.
     *
     * If you group service interfaces by functional area (for example,
     * in packages com.xyz.someapp.abc.service and com.xyz.someapp.def.service) then
     * the pointcut expression "execution(* com.xyz.someapp..service.*.*(..))"
     * could be used instead.
     *
     * Alternatively, you can write the expression using the 'bean'
     * PCD, like so "bean(*Service)". (This assumes that you have
     * named your Spring service beans in a consistent fashion.)
     */
    @Pointcut("execution(* com.xyz.someapp..service.*.*(..))")
    public void businessService() {}

    /**
     * A data access operation is the execution of any method defined on a
     * dao interface. This definition assumes that interfaces are placed in the
     * "dao" package, and that implementation types are in sub-packages.
     */
    @Pointcut("execution(* com.xyz.someapp.dao.*.*(..))")
    public void dataAccessOperation() {}

}
```
在这样的切面定义的切入点可以被引用到你需要切入点表达式的任何地方; 例如, 要使服务层具有事务性, 可以编写为
```
<aop:config>
    <aop:advisor
        pointcut="com.xyz.someapp.SystemArchitecture.businessService()"
        advice-ref="tx-advice"/>
</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```
`<aopconfig>` 和 `<aopadvisor>` 元素将在 [第 11.3 节 "基于模式的 AOP 支持"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-schema) 中讨论; [第 17 章 "事务管理"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/transaction.html) 中讨论了事务元素

##### 示例
Spring AOP 用户可能最常使用执行切入点指示符, 执行表达式的格式为:
```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)
```
除返回类型模式 (上面的代码段中的 `ret-type-pattern`), 名称模式和参数模式之外的所有部分都是可选的; 返回类型模式确定方法的返回类型必须是什么才能匹配连接点; 最常见的是, 你将使用 `*` 作为返回类型模式, 它匹配任何返回类型; 仅当方法返回给定类型时, 完全限定类型名称才匹配; 名称模式与方法名称匹配; 你可以使用 `*` 通配符作为名称模式的全部或部分; 如果指定声明类型模式, 则包括尾随 `.` 将其加入名称模式组件; 参数模式稍微复杂一些: `()` 匹配不带参数的方法, 而 `(..)` 匹配任意数量的参数 (零或更多); 模式 `(*)` 匹配采用任何类型的一个参数的方法, `(*, String)` 匹配采用两个参数的方法, 第一个可以是任何类型, 第二个必须是 `String`; 有关更多信息, 请参阅 AspectJ编程指南的 [语言语义](https://www.eclipse.org/aspectj/doc/released/progguide/semantics-pointcuts.html) 部分  
下面给出了常见切入点表达式的一些示例
- 执行任何公共方法
```
execution(public * *(..))
```
- 执行任何以 `set` 开头的方法
```
execution(* set*(..))
```
- 执行 `AccountService` 接口定义的任何方法
```
execution(* com.xyz.service.AccountService.*(..))
```
- 执行 service 包中定义的任何方法
```
execution(* com.xyz.service.*.*(..))
```
- 执行 service 包或子包中定义的任何方法
```
execution(* com.xyz.service..*.*(..))
```
- service 包中的任何连接点 (仅在 Spring AOP 中执行方法)
```
within(com.xyz.service.*)
```
- service 包或子包中的任何连接点 (仅在 Spring AOP 中执行方法)
```
within(com.xyz.service..*)
```
- 代理实现 `AccountService` 接口的任何连接点 (仅在 Spring AOP 中执行方法)
```
this(com.xyz.service.AccountService)
```
>`this` 更常用于绑定形式: 请参阅以下有关如何在通知体中提供代理对象通知
- 目标对象实现 `AccountService` 接口的任何连接点 (仅在 Spring AOP 中执行方法)
```
target(com.xyz.service.AccountService)
```
>`target` 更常用于绑定形式: 请参阅以下有关如何在通知体中提供目标对象的通知
- 任何连接点 (仅在 Spring AOP 中执行的方法), 它接受一个参数, 并且在运行时传递的参数是 `Serializable`
```
args(java.io.Serializable)
```
>`args` 更常用于绑定形式: 请参阅以下有关如何在通知体中提供代理对象通知

请注意, 此示例中给出的切入点与 `execution(* *(java.io.Serializable))` 不同：如果在运行时传递的参数是 `Serializable` 则 args 版本匹配, 如果方法签名声明单个参数类型 `Serializable` 则执行版本匹配
- 任何连接点 (仅在 Spring AOP 中执行的方法), 其中目标对象具有 `@Transactional` 注解
```
@target(org.springframework.transaction.annotation.Transactional)
```
>`@target` 更常用于绑定形式: 请参阅以下有关如何在通知体中提供代理对象通知

- 任何连接点 (仅在 Spring AOP 中执行方法), 其中目标对象的声明类型具有 `@Transactional` 注解
```
@within(org.springframework.transaction.annotation.Transactional)
```
>`@within` 更常用于绑定形式: 请参阅以下有关如何在通知体中提供代理对象通知

- 任何连接点 (仅在 Spring AOP 中执行方法), 其中执行方法具有 `@Transactional` 注解
```
@annotation(org.springframework.transaction.annotation.Transactional)
```
>`@annotation` 更常用于绑定形式: 请参阅以下有关如何在通知体中提供代理对象通知

- 任何连接点 (仅在 Spring AOP 中执行的方法), 它接受一个参数并且其运行时类型具有 `@Classified` 注解
```
@args(com.xyz.security.Classified)
```
>`@args` 更常用于绑定形式: 请参阅以下有关如何在通知体中提供代理对象通知

- 名为 `tradeService` 的 Spring bean 上的任何连接点 (仅在 Spring AOP 中执行方法)
```
bean(tradeService)
```
- Spring bean 上的任何连接点 (仅在 Spring AOP 中执行方法), 其名称与通配符表达式 `*Service` 匹配
```
bean(*Service)
```

##### 写出好的切入点
在编译期间, AspectJ 处理切入点以尝试和优化匹配性能; 检查代码并确定每个连接点是否 (静态地或动态地) 匹配给定切入点是一个代价高昂的过程; (动态匹配意味着无法通过静态分析完全确定匹配, 并且将在代码中放置测试以确定代码运行时是否存在实际匹配); 在第一次遇到切入点声明时, AspectJ 会将其重写为匹配过程的最佳形式; 这是什么意思? 基本上切入点在 `DNF (析取范式)` 中重写, 并且切入点的组件被排序, 以便首先检查那些评估成本更低的组件; 这意味着你不必担心理解各种切入点指示符的性能, 并且可以在切入点声明中以任何顺序提供它们  
但是, AspectJ 只能处理它所说的内容, 并且为了获得最佳匹配性能, 你应该考虑它们想要实现的目标, 并在定义中尽可能缩小匹配的搜索空间; 现有的指示符自然分为三组: `kinded, scoping, context`
- Kinded 指示符是选择特定类型的连接点的指示符; 例如: `execution, get, set, call, handler`
- Scoping 界定指示符是那些选择一组感兴趣的连接点 (可能是多种类型) 的指示符; 例如: `within, withincode`
- Contextual 指示符是基于上下文匹配 (并且可选地绑定) 的指示符; 例如: `this, target, @annotation`

一个写得很好的切入点应该尝试包括至少前两种类型 (kinded 和 scoping), 而如果希望基于连接点上下文匹配, 则可以包括上下文指示符, 或者绑定该上下文以在通知中使用; 仅提供一个 `kinded` 指示符或仅提供 `context` 指示符将起作用, 但由于所有额外的处理和分析, 可能会影响编织性能 (使用的时间和内存); `scoping` 指示符非常快速匹配, 它们的使用意味着 AspectJ 可以非常快速地解除不应该进一步处理的连接点组 --- 这就是为什么如果可能的话, 一个好的切入点应该总是包含一个

#### 声明通知
通知与切入点表达式相关联, 并在切入点匹配的方法执行之前, 之后或周围运行; 切入点表达式可以是对命名切入点的简单引用, 也可以是在适当位置声明的切入点表达式

##### Before advice
在切面中使用 `@Before` 注解声明前置通知
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

}
```
如果使用就地切入点表达式, 我们可以将上面的示例重写为
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("execution(* com.xyz.myapp.dao.*.*(..))")
    public void doAccessCheck() {
        // ...
    }

}
```
##### 后置返回通知
后置返回通知匹配的方法在正常返回后执行, 它是使用 `@AfterReturning` 注解声明的
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

}
```
>注意: 当然可以在同一切面内有多个通知声明和其他成员; 我们只是在这些例子中展示了一个通知声明, 专注于当时正在讨论的问题

有时你需要在通知体中访问返回的实际值, 你可以使用 `@AfterReturning` 的形式来绑定返回值
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }

}
```
`returns` 属性中使用的名称必须与通知方法中的参数名称相对应; 当方法执行返回时, 返回值将作为相应的参数值传递给通知方法; 返回子句还将匹配限制为仅返回指定类型的值的那些方法执行 (在这种情况下, Object 将匹配任何返回值)  
请注意, 在使用返回后的通知时, 无法返回完全不同的引用

##### 后置异常通知
后置异常通知运行时, 匹配的方法执行通过抛出异常退出, 它是使用 `@AfterThrowing` 注解声明的
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doRecoveryActions() {
        // ...
    }

}
```
通常, 你希望仅在抛出给定类型的异常时才运行通知, 并且你还经常需要访问通知体中的抛出异常; 使用 `throwing` 属性来限制匹配 (如果需要, 否则使用 `Throwable` 作为异常类型) 并将抛出的异常绑定到通知参数
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {
        // ...
    }

}
```
`throw` 属性中使用的名称必须与通知方法中的参数名称相对应; 当通过抛出异常退出方法时, 异常将作为相应的参数值传递给通知方法; `throw` 子句还将匹配仅限于那些抛出指定类型异常的方法执行 (在本例中为 `DataAccessException`)

##### 后置 (最终) 通知
在后置 (最终) 通知运行时, 匹配的方法执行退出; 它是使用 `@After` 注解声明的; 在通知必须准备好处理正常和异常返回条件; 它通常用于释放资源等
```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.After;

@Aspect
public class AfterFinallyExample {

    @After("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doReleaseLock() {
        // ...
    }

}
```

##### 环绕通知
最后一种通知是环绕通知, 周围的通知围绕匹配的方法执行运行; 它有机会在方法执行之前和之后完成工作, 并确定方法实际上何时, 如何, 甚至是否实际执行; 如果你需要以线程安全的方式 (例如, 启动和停止计时器) 在方法执行之前和之后共享状态, 则经常使用环绕通知; 始终使用符合你要求的最小的通知形式 (即如果简单, 能使用前置通知就不要使用环绕通知)  
使用 `@Around` 注解声明环绕通知, 通知方法的第一个参数必须是 `ProceedingJoinPoint` 类型; 在通知的主体内, 在 `ProceedingJoinPoint` 上调用 `proceed()` 会导致执行基础方法; `proceed()` 也可以调用传递给 `Object[]` --- 数组中的值将用作方法执行的参数
>使用 `Object[]` 调用时, `proceed()` 的行为与由 AspectJ 编译器编译的环绕通知的行为略有不同; 对于使用传统 AspectJ 语言编写的环绕通知, 传递给 `proceed()` 的参数数量必须与传递给环绕通知的参数数量 (不是基础连接点所采用的参数数量) 相匹配, 并且传递给的值继续给定的参数位置取代了值绑定到的实体的连接点的原始值 (如果现在没有意义, 不要担心!); Spring 采用的方法更简单, 更好地匹配其基于代理的, 仅执行语义; 如果要编译为 Spring 编写的 `@AspectJ` 切面并使用带有 AspectJ 编译器和 weaver 的参数继续, 则只需要知道这种差异; 有一种方法可以编写这样的切面, 这些切面在 Spring AOP 和 AspectJ 上都是 100% 兼容的, 这将在下面的通知参数部分中讨论

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class AroundExample {

    @Around("com.xyz.myapp.SystemArchitecture.businessService()")
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // start stopwatch
        Object retVal = pjp.proceed();
        // stop stopwatch
        return retVal;
    }

}
```
环绕通知返回的值将是方法调用者看到的返回值; 例如, 一个简单的缓存方面可以从缓存中返回一个值 (如果有的话), 如果没有则调用 `proceed()`; 请注意, 可以在环绕建议的主体内调用一次, 多次, 或者根本不调用, 所有这些都是合乎逻辑的

##### 通知参数
Spring 提供全类型的通知 --- 意味着你在通知签名中声明了所需的参数 (正如我们在上面看到的返回和抛出示例所示), 而不是一直使用 `Object[]` 数组; 我们将看到如何在一瞬间为通知主体提供参数和其他上下文值; 首先让我们来看看如何编写通用通知, 以便了解通知目前的方法
##### 访问当前的连接点


>**参考:**
[@AspectJ support](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj)
