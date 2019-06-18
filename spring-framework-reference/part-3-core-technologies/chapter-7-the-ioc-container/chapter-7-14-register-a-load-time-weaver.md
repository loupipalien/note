### 注册一个 LoadTimeWeaver
在将类加载到 Java 虚拟机 (JVM) 时, Spring 使用 `LoadTimeWeaver` 动态转换类  
要启用加载时编织, 请将 `@EnableLoadTimeWeaving` 添加到其中一个 `@Configuration` 类中
```
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```
或者, 对于XML配置, 请使用 `context:load-time-weaver` 元素
```
<beans>
    <context:load-time-weaver/>
</beans>
```
为 `ApplicationContext` 配置后; `ApplicationContext` 中的任何 bean 都可以实现 `LoadTimeWeaverAware`, 从而接收对 `load-time weaver` 实例的引用; 这与 Spring 的 JPA 支持结合使用特别有用, 其中 JPA 类转换可能需要加载时编织; 有关更多详细信息, 请参阅
`LocalContainerEntityManagerFactoryBean` 的 javadocs; 有关 `AspectJ` 加载时编织的更多信息, 请参见 [第 11.8.4 节 "在 Spring Framework 中使用 `AspectJ` 进行加载时编织"](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/aop.html#aop-aj-ltw)

>**参考:**  
[Registering a LoadTimeWeaver](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#context-load-time-weaver)
