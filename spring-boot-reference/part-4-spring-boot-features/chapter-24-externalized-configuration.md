### 外部化配置
Spring Boot 允许你外部化配置, 以便你可以在不同的环境中使用相同的应用程序代码; 你可以使用属性文件, YAML 文件, 环境变量和命令行参数来外部化配置; 可以使用 `@Value` 注解将属性值直接注入到 bean 中, 通过 Spring 的 `Environment` 抽象访问, 或者通过 `@ConfigurationProperties` 绑定到结构化对象  
Spring Boot 使用一个非常特殊的 `PropertySource` 顺序, 旨在允许合理地覆盖值; 按以下顺序考虑属性  
1. 在你的家目录下的 [Devtools 全局设置属性](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/using-boot-devtools.html#using-boot-devtools-globalsettings) (`~/.spring-boot-devtools.properties`, 当 devtools 处于激活时)  
2. 注解在你的测试上的注解 `@TestPropertySource`
3. 在你的测试上的 `properties` 属性; 可在 [`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/api/org/springframework/boot/test/context/SpringBootTest.html) 上使用, 以及 [用于测试应用程序特定片段的测试注解](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)
4. 命令行参数
5. 从 `SPRING_APPLICATION_JSON` 中来的属性 (嵌入在环境变量或系统属性中的内联 JSON)
6. `ServletConfig` 初始化参数
7. `ServletContetx` 初始化参数
8. 从 `java:comp/env` 中来的 JNDI 属性
9. Java 系统属性 (`System.getProperties()`)
10. OS 环境变量
11. 只有 `random.*` 属性的 `RandomValuePropertySource`
12. 在你的打包 jar 外的 [特定配置的应用程序属性](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config-profile-specific-properties) (`application-{profile}.properties` 和 YAML 变体)
13. 在你的打包 jar 内的 [特定配置的应用程序属性](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config-profile-specific-properties) (`application-{profile}.properties` 和 YAML 变体)
14. 在你的打包 jar 外的应用程序属性 (`application.properties` 和 YAML 变体)
15. 在你的打包 jar 内的应用程序属性 (`application.properties` 和 YAML 变体)
16. 注解在你的 `@Configuration` 注解的类上的 `@PropertySource`
17. 默认属性 (通过设置 `SpringApplication.setDefaultProperties` 指定)

以下提供一个具体示例, 假设你开发了一个使用 `name` 属性的 `@Component`, 如以下所示
```
import org.springframework.stereotype.*;
import org.springframework.beans.factory.annotation.*;

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...

}
```
在应用程序类路径上 (例如, 在jar中), 你可以拥有一个 `application.properties` 文件, 该文件为 `name` 提供合理的默认属性值; 在新环境中运行时, 可以在 jar 之外提供覆盖名称的 `application.properties` 文件; 对于一次性测试, 你可以使用特定的命令行开关启动 (例如, `java -jar app.jar --name ="Spring"`)
>可以在命令行上使用环境变量提供 `SPRING_APPLICATION_JSON` 属性; 例如, 在 UN*X shell 你可以使用以下命令行
>`$ SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar`
>在上述的示例中, 你最终在 Spring 环境中使用了 `acme.name = test`; 你还可以在系统属性中将 JSON 作为 `spring.application.json` 提供, 如以下所示
>`java -Dspring.application.json='{"name":"test"}' -jar myapp.jar`
>你还可以使用命令行参数提供 JSON, 如以下所示
>`java -jar myapp.jar --spring.application.json='{"name":"test"}'`
>你还可以将 JSON 作为 JNDI 变量提供, 如下所示: `java:comp/env/spring.application.json`

#### 配置随机值
`RandomValuePropertySource` 对于注入随机值非常有用 (例如, 在密码或者测试用例中使用); 它可以产生整型, 长整形, uuid, 字符串, 如以下所示
```
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```
`random.int*` 语法是 `OPEN value (,max) CLOSE`, 其中 `OPEN, CLOSE` 是任何字符, `value, max` 是整数; 如果提供了 `max`, 则 `value` 是最小值, 并且 `max` 是最大值 (不包括)

#### 访问命令行参数
默认情况, `SpringApplication` 将任何命令行选项参数 (即以 `--` 开头的参数, 例如 `--server.port = 9000`) 转换为 `property` 并将它们添加到 Spring `Environment` 中; 如前所述, 命令行属性始终优先于其他属性源  
如果你不希望将命令行属性添加到 `Environment`, 可以使用 `SpringApplication.setAddCommandLineProperties(false)` 禁用它们

#### 应用程序属性文件
`SpringApplication` 从以下位置的 `application.properties` 文件加载属性, 并将它们添加到 Spring `Environment` 中
1. 当前目录的 `/config` 子目录
2. 当前目录
3. 类路径下的 `/config` 包
4. 类路径的根

列表按优先级排序 (在列表中较高位置定义的属性将覆盖在较低位置中定义的属性)

>**参考:**  
[Externalized Configuration](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config)
