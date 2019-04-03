### 测试
Spring Boot 提供了许多实用工具和注解来帮助你测试应用程序; 测试支持由两个模块提供: `spring-boot-test` 包含核心项, `spring-boot-test-autoconfigure` 支持测试的自动配置
大多数开发人员使用 `spring-boot-starter-test` 启动器来导入 Spring Boot 测试模块以及 JUnit, AssertJ, Hamcrest 以及许多其他有用的库

#### 测试范围依赖
`spring-boot-starter-test` 启动器 (在 `test scope`) 包含以下提供库
- [JUnit](http://junit.org/): Java 应用程序单元测试的事实标准
- [Spring Test](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/testing.html#integration-testing) 和 Spring Boot Test: 支持 Spring Boot 应用程序的实用工具和集成测试
- [AssertJ](https://joel-costigliola.github.io/assertj/): 一个流式的断言库
- [Hamcrest](http://hamcrest.org/JavaHamcrest/): 一个匹配器对象库 (也称为约束或谓词)
- [Mockito](http://mockito.org/): 一个 Java 模拟框架
- [JSONassert](https://github.com/skyscreamer/JSONassert): 一个 JSON 的断言库
- [JsonPath](https://github.com/jayway/JsonPath): JSON 的 XPath

我们通常发现这些常用库在编写测试时很有用; 如果这些库不符合你的需求, 你可以添加其他测试依赖

#### 测试 Spring 应用程序
依赖注入的一个主要优点是它可以使你的代码更容易进行单元测试; 你可以使用 `new` 运算符实例化对象, 甚至不涉及 Spring; 你也可以使用模拟对象而不是真正的依赖  
通常, 你除了需要单元测试还需要开始集成测试 (使用一个 Spring `ApplicationContext`); 在不需要部署应用程序或需要连接到其他基础架构的情况下执行集成测试非常有用  
Spring Framework 包含一个用于此类集成测试的专用测试模块; 你可以直接声明 `org.springframework:spring-test` 依赖或使用 `spring-boot-starter-test` 启动器来传递此依赖  
如果你之前没有使用过 `spring-test` 模块, 那么首先应阅读 Spring Framework 参考文档的 [相关部分](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/testing.html#testing)

#### 测试 Spring Boot 应用程序
Spring Boot 应用程序是一个 Spring `ApplicationContext`, 因此与你通常使用的 Spring 上下文之外, 测试时并没有什么特别要做的
>只有在使用 SpringApplication 创建它时, Spring Boot 的外部属性, 日志记录, 以及其他功能会默认安装在上下文中

Spring Boot 提供了一个 `@SpringBootTest` 注解, 当你需要 Spring Boot 功能时, 它可以用作标准 `spring-test` `@ContextConfiguration` 注解替代项; 注解通过 [`SpringApplication` 创建测试中使用的 `ApplicationContext`](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-testing-spring-boot-applications-detecting-config) 来工作; 除了 `@SpringBootTest`之外, 还提供了许多其他注解来测试应用程序中 [更具体的项](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)
>如果你使用的是 JUnit 4, 请不要忘记将 `@RunWith(SpringRunner.class)` 添加到你的测试类上, 否则注解将会被忽略; 如果你使用的是 JUnit 5, 则无需将等效的 `@ExtendWith(SpringExtension)` 添加在 `@SpringBootTest` 以及 `@...Test` 注解已经使用的类上了

默认的, `@SpringBootTest` 不会启动服务器; 你可以使用 `@SpringBootTest` 的 `webEnvironment` 属性来进一步细化测试的运行方式
- MOCK（默认）：加载 Web ApplicationContext 并提供模拟 Web 环境; 使用此注解时, 不会启动嵌入式服务器; 如果类路径上没有 Web 环境, 则此模式将透明地回退到创建常规非 Web ApplicationContext; 它可以与 [`@AutoConfigureMockMvc` 或 `@AutoConfigureWebTestClient`](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-with-mock-environment) 结合使用, 以进行基于模拟的 Web 应用程序测试
- RANDOM_PORT: 加载 `WebServerApplicationContext` 并提供真实的 Web 环境; 嵌入式服务器启动并在随机端口上侦听
- DEFINED_PORT: 加载 `WebServerApplicationContext` 并提供真实的 Web 环境; 嵌入式服务器启动并侦听定义的端口 (来自你的 `application.properties`) 或默认端口 `8080`
- NONE: 使用 `SpringApplication` 加载 `ApplicationContext`, 但不提供任何 Web 环境 (模拟或其他)

>如果你的测试注解了 `@Transactional`, 则默认情况下会在每个测试方法的末尾回滚事务; 但由于使用 `RANDOM_PORT` 或 `DEFINED_PORT` 这种配置隐式提供了一个真正的 servlet 环境, 因此 HTTP 客户端和服务器在不同的线程中运行, 因此在单独的事务中运行; 在这种情况下, 在服务器上启动的任何事务都不会回滚
>如果你的应用程序需使用不同的服务器端口, 带有 `webEnvironment = WebEnvironment.RANDOM_PORT` 的 `@SpringBootTest` 将在单独的随机端口上启动服务器

##### 探测 Web 应用程序类型
如果 Spring MVC 可用, 基于 MVC 的常规应用程序上下文会被配置; 如果你只有 Spring WebFlux, 我们会检测到并配置基于 WebFlux 的应用程序上下文  
如果两者都存在, 则 Spring MVC 优先; 如果要在此方案中测试响应式 Web 应用程序, 则必须设置 `spring.main.web-application-type` 属性
```
@RunWith(SpringRunner.class)
@SpringBootTest(properties = "spring.main.web-application-type=reactive")
public class MyWebFluxTests { ... }
```

##### 探测测试配置
如果你熟悉 Spring Test Framework, 则可能习惯使用 `@ContextConfiguration(classes = ...)` 来指定要加载的 Spring `@Configuration`; 或者, 你可以在测试中使用嵌套 `@Configuration` 的类  
在测试 Spring Boot 应用程序时, 通常不需要这样做; Spring Boot 的 `@*Test` 注解会在你未明确定义配置时自动搜索主要配置
搜索算法从包含测试的包开始工作, 直到找到使用 `@SpringBootApplication` 或 `@SpringBootConfiguration` 注解的类; 只要你以合理的方式构建代码, 通常会找到主要配置
>如果使用 [测试注解来测试应用程序的更具体的片段](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests), 则应避免在 [main 方法的应用程序类](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-user-configuration) 中添加特定于特定区域的配置  
`@SpringBootApplication` 的基础组件扫描配置定义了排除过滤器, 用于确保切片按预期工作; 如果在 `@SpringBootApplication` 注解的类上使用显式的 `@ComponentScan` 指令, 请注意这些过滤器将被禁用; 如果你正在使用切片, 则应再次定义它们

如果要自定义主要配置, 可以使用注解的 `@TestConfiguration` 的类; 与用于代替应用程序的主要配置的 `@Configuration` 类不同, 注解的 `@TestConfiguration` 类是用于增添到应用程序的主要配置中
> Spring 的测试框架在测试之间缓存应用程序上下文; 因此, 只要你的测试共享相同的配置 (无论如何发现), 加载上下文的潜在耗时过程只发生一次

##### 排除测试配置
如果你的应用程序使用组件扫描 (例如, 如果你使用 `@SpringBootApplication` 或 `@ComponentScan`), 你可能会偶然发现仅为特定测试创建的顶层配置类会在任何地方被获取  
正如我们 [之前所见](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-testing-spring-boot-applications-detecting-config), `@TestConfiguration` 可用于测试的内部类以自定义主要配置; 当置于顶层类时, `@TestConfiguration` 意味着不应通过扫描获取 `src/test/java` 中的类; 然后, 你可以在需要的位置显式导入该类, 如下所示
```
@RunWith(SpringRunner.class)
@SpringBootTest
@Import(MyTestsConfiguration.class)
public class MyTests {

	@Test
	public void exampleTest() {
		...
	}

}
```
>如果直接使用  `@ComponentScan` (不通过 `@SpringBootApplication`), 则需要使用其注册 `TypeExcludeFilter`; 详见 [Javadoc](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/api/org/springframework/boot/context/TypeExcludeFilter.html)

##### 使用 mock 环境测试











>**参考:**
[Testing](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-testing)
