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

>**参考:**
[@AspectJ support](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj)
