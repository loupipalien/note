### 编程式创建 AspectJ 代理
除了使用 `<aop:config>` 或 `<aop:aspectj-autoproxy>` 在配置中声明方面之外, 还可以通过编程方式创建通知目标对象的代理; 有关 Spring 的 AOP API 的完整详细信息, 请参阅下一章; 在这里, 我们希望关注使用 @AspectJ 切面自动创建代理的能力  
`org.springframework.aop.aspectj.annotation.AspectJProxyFactory` 类可用于为一个或多个 @AspectJ 切面建议的目标对象创建代理; 此类的基本用法非常简单, 如下所示; 有关完整信息请参阅 javadocs
```
// create a factory that can generate a proxy for the given target object
AspectJProxyFactory factory = new AspectJProxyFactory(targetObject);

// add an aspect, the class must be an @AspectJ aspect
// you can call this as many times as you need with different aspects
factory.addAspect(SecurityManager.class);

// you can also add existing aspect instances, the type of the object supplied must be an @AspectJ aspect
factory.addAspect(usageTracker);

// now get the proxy object...
MyInterfaceType proxy = factory.getProxy();
```
>**参考:**  
[Programmatic creation of @AspectJ Proxies](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-aspectj-programmatic)
