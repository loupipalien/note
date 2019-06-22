### 介绍
面向切面编程 (AOP) 通过提供另一种思考程序结构的方式来补充面向对象编程 (OOP); OOP 中模块化的关键单元是类, 而在 AOP 中, 模块化单元是切面; 切面实现了诸如跨越多种类型和对象的事务管理之类的关注点的模块化 (这种担忧通常被称为 AOP 文献中的横切关注点)  
Spring 的一个关键组件是 AOP 框架; 虽然 Spring IoC 容器不依赖于 AOP, 如果你不想使用 AOP 则意味着你不需要使用 AOP, 但 AOP 补充了 Spring IoC 以提供非常强大的中间件解决方案  
```
Spring 2.0+ AOP
Spring 2.0 引入了一种使用 [基于模式的方法](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-schema) 或 [@AspectJ 注解样式](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-ataspectj) 编写自定义方面的更简单, 更强大的方法; 这两种样式都提供完全类型的通知和 AspectJ 切入点语言的使用, 同时仍然使用 Spring AOP 进行编织  
本章将讨论 Spring 2.0+ 模式和基于 @AspectJ 的 AOP 支持; [接下来一章](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop-api.html) 将讨论较低级别的 AOP 支持, 如 Spring 1.2 应用程序中常见的那样
```
AOP 在 Spring Framework 中用于
- 提供声明性企业服务, 尤其是作为 EJB 声明性服务的替代品; 最重要的此类服务是 [声明式事务管理](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/transaction.html#transaction-declarative)
- 允许用户实现自定义方面, 用 AOP 补充他们使用的 OOP

>如果你只对通用声明性服务或其他预先打包的声明性中间件服务 (如池) 感兴趣, 则无需直接使用 Spring AOP, 并且可以跳过本章的大部分内容

#### AOP 概念
让我们首先定义一些 AOP 的核心概念和术语; 这些术语不是特定于 Spring 的... 不幸的是, AOP 术语不是特别直观; 但是如果 Spring 使用自己的术语, 那将更加令人困惑
- 切面 (Aspect): 跨越多个类别的关注点的模块化; 事务管理是企业 Java 应用程序中横切关注点的一个很好的例子; 在 Spring AOP 中, 切面是使用常规类 (基于模式的方法) 或使用 `@Aspect` 注解 (`@AspectJ` 样式) 的常规类实现的
- 连接点 (Join point): 执行程序期间的一个点, 例如执行方法或处理异常; 在 Spring AOP 中, 连接点始终表示方法执行
- 通知 (Advice): 某个切面在特定连接点处采取的操作; 不同类型的建议包括 "around", "before", "after" 等通知 (通知类型将在下面讨论) 许多 AOP 框架 (包括 Spring) 将通知建模为拦截器, 在连接点周围维护一系列拦截器
- 切入点 (Pointcut): 与连接点匹配的谓词; 通知与切入点表达式相关联, 并在切入点匹配的任何连接点处运行 (例如, 执行具有特定名称的方法); 由切入点表达式匹配的连接点的概念是 AOP 的核心, Spring 默认使用 `AspectJ` 切入点表达式语言  
- 引言 (Introduction): 在类型上声明额外方法或字段; Spring AOP 允许你向任何通知的对象引入新接口 (以及相应的实现); 例如, 你可以使用引言使 bean 实现 `IsModified` 接口, 以简化缓存 (引言被称为 AspectJ 社区中的类型间声明)
- 目标对象 (Target object): 一个或多个切面通知的对象, 也称为通知对象; 由于 Spring AOP 是使用运行时代理实现的, 因此该对象始终是代理对象
- AOP 代理: 由 AOP 框架创建的对象, 用于实现切面契约 (通知方法执行等); 在 Spring Framework 中, AOP 代理将是 JDK 动态代理或 CGLIB 代理
- 织入 (Weaving): 将切面与其他应用程序类型或对象链接以创建通知对象; 这可以在编译时 (例如, 使用 AspectJ 编译器) 加载时间或在运行时完成; 与其他纯 Java AOP 框架一样, Spring AOP 在运行时执行编织

通知类型
- 前置通知 (Before advice): 在连接点之前执行但不能阻止执行流程进入连接点的通知 (除非它引发异常)
- 后置返回通知 (After returning advide): 在连接点正常完成后要执行的通知: 例如, 如果方法返回而不抛出异常
- 后置异常通知 (After throwing advice): 如果方法通过抛出异常退出, 则执行通知
- 后置通知 (After advice): 无论连接点退出的方式 (正常或异常返回), 都要执行的通知
- 环绕通知 (Around advice): 围绕连接点的通知, 例如方法调用; 这是最有力的通知; around 通知可以在方法调用之前和之后执行自定义行为; 它还负责选择是继续处理连接点还是通过返回自己的返回值或抛出异常来短路被通知方法的执行

环绕通知是最通用的通知; 由于 Spring AOP (如 AspectJ) 提供了全方位的建议类型, 因此我们建议你使用可以实现所需行为的最小通知类型; 例如, 如果你只需要使用方法的返回值更新缓存, 那么最好是实后置返回通知现而不是环绕通知, 尽管环绕通知可以完成同样的事情; 使用最具体的通知类型可以提供更简单的编程模型, 减少错误的可能性; 例如, 你不需要在用于环绕通知的 `JoinPoint` 上调用 `proceed()` 方法, 因为无法调用它  
在 Spring 2.0 中, 所有通知的参数都是静态类型的, 因此你可以使用相应类型的通知参数 (例如, 方法执行的返回值的类型) 而不是 `Object` 数组  
通过切入点匹配的连接点的概念是 AOP 的关键, 它将其与仅提供拦截的旧技术区分开来; 切入点使得建议可以独立于面向对象的层次结构进行定向; 例如, 提供声明式事务管理的环绕通知可以应用于跨越多个对象的一组方法 (例如服务层中的所有业务操作)

#### Spring AOP 的功能和目标
Spring AOP 是用纯 Java 实现的, 不需要特殊的编译过; Spring AOP 不需要控制类加载器层次结构, 因此适合在 `Servlet` 容器或应用程序服务器中使用  
Spring AOP 目前仅支持方法执行连接点 (建议在 Spring bean 上执行方法); 虽然可以在不破坏核心 Spring AOP API 的情况下添加对字段拦截的支持, 但未实现字段拦截; 如果你需要建议字段访问和更新连接点, 请考虑使用 AspectJ 等语言  
Spring AOP 的 AOP 方法与大多数其他 AOP 框架的方法不同; 目的不是提供最完整的 AOP 实现 (尽管 Spring AOP 非常强大); 它是在 AOP 实现和 Spring IoC 之间提供紧密集成, 以帮助解决企业应用程序中的常见问题  
因此, 例如 Spring Framework 的 AOP 功能通常与 Spring IoC 容器一起使用; 使用普通 bean 定义语法配置方面 (尽管这允许强大的 "自动代理"功能): 这是与其他 AOP 实现的重要区别; 使用 Spring AOP 时, 你可以轻松或高效地执行某些操作, 例如建议非常细粒度的对象 (例如域对象): 在这种情况下, AspectJ 是最佳选择; 但是, 我们的经验是 Spring AOP 为适合 AOP 的企业 Java 应用程序中的大多数问题提供了出色的解决方案  
Spring AOP 永远不会努力与 AspectJ 竞争, 以提供全面的 AOP 解决方案; 我们相信像 Spring AOP 这样的基于代理的框架和像 AspectJ 这样的完整框架都很有价值, 而且它们是互补的而不是竞争; Spring 将 Spring AOP 和 IoC 与 AspectJ 无缝集成, 以便在基于 Spring 的一致应用程序架构中满足 AOP 的所有使用需求; 此集成不会影响 Spring AOP API 或 AOP Alliance API: Spring AOP 保持向后兼容; 有关 Spring AOP API 的讨论, 请参阅 [后续章节](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop-api.html)
>Spring 框架的核心原则之一是非侵入性; 这是一个想法, 你不应该被迫在你的业务/领域模型中引入特定于框架的类和接口; 但是, 在某些地方, Spring Framework 确实为你提供了将 Spring Framework 特定的依赖项引入代码库的选项; 为你提供这些选项的基本原理是因为在某些情况下, 它可能更容易阅读或编写某些特定的部分以这种方式的功能; Spring Framework (几乎) 总是为你提供选择: 你可以自由决定哪种选项最适合您的特定用例或场景
与本章相关的一个选择是选择哪种 AOP 框架 (以及哪种 AOP 样式), 你可以选择 AspectJ 或 Spring AOP, 也可以选择 `@AspectJ` 注解样式方法或 Spring XML 配置样式方法; 本章选择首先介绍 `@AspectJ` 风格的方法, 这一事实不应被视为 Spring 团队倾向于采用 Spring XML配置风格之上使用 `@AspectJ` 注解风格  
请参见 [第 11.4 节 "选择使用哪种 AOP 声明样式"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-choosing), 以更全面地讨论每种样式的原因和原因

#### AOP 代理
Spring AOP 默认使用 AOP 代理的标准 JDK 动态代理; 这使得任何接口 (或接口集) 都可以被代理  
Spring AOP 也可以使用 CGLIB 代理; 这是代理类而不是接口所必需的; 如果业务对象未实现接口, 则默认使用 CGLIB; 因为优良的做法是编程接口而不是类; 业务类通常会实现一个或多个业务接口, 可以强制使用 CGLIB, 在那些需要通知未在接口上声明的方法, 或者需要将代理对象作为具体类型传递给方法的情况下 (很少见)  
掌握 Spring AOP 是基于代理的这一事实非常重要; 请参见 [第 11.6.1 节 "了解 AOP 代理"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-understanding-aop-proxies), 以全面了解此实现细节的实际含义

>**参考:**
[Introduction](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/htmlsingle/#aop-introduction)
