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
>你也可以使用 [YAML('.yml')](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config-yaml) 文件作为代替 `properties` 文件

如果你不喜欢 `application.properties` 作为配置文件名, 则可以通过指定 `spring.config.name` 环境属性来切换到另一个文件名; 你还可以使用 `spring.config.location` 环境属性 (以逗号分隔的目录位置或文件路径列表) 来引用一个确切的位置; 以下示例展示如何指定其他文件名
```
$ java -jar myproject.jar --spring.config.name=myproject
```
以下示例展示如何指定两个配置文件
```
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
```
>`spring.config.name` 和 `spring.config.location` 很早就用来确定必须加载哪些文件, 因此必须将它们定义为环境属性 (通常是 OS 环境变量, 系统属性或命令行参数)

如果 `spring.config.location` 包含目录 (而不是文件), 则它们应以 `/` 结尾 (并且在运行时, 在加载之前附加从 `spring.config.name` 生成的名称, 包括特定于配置文件的文件名); `spring.config.location` 中指定的文件按原样使用, 不支持特定于配置文件的变体, 并且被任何特定于配置文件的属性覆盖  
以相反的顺序搜索配置位置; 默认情况下, 配置的位置是 `classpath:/, classpath:/config/, file:./, file:./config/`; 生成的搜索顺序如下:
1. `file:./config/`
2. `file:./`
3. `classpath:/config/`
4. `classpath:/`

使用 `spring.config.location` 配置自定义配置位置时, 它们将替换默认位置; 例如, 如果 `spring.config.location` 配置了值 `classpath:/custom-config/, file:./custom-config/`, 搜索顺序将变为以下:
1. `classpath:/custom-config/`
2. `file:./custom-config/`

或者, 当使用 `spring.config.additional-location` 配置自定义配置位置时, 除默认位置外, 还会使用它们; 在默认位置之前搜索其他位置; 例如, 如果配置了 `classpath:/custom-config/, file:./custom-config/` 的其他位置, 则搜索顺序将变为以下:
1. `classpath:/custom-config/`
2. `file:./custom-config/`
3. `file:./config/`
4. `file:./`
5. `classpath:/config/`
6. `classpath:/`

此搜索顺序允许你在一个配置文件中指定默认值, 然后有选择地覆盖另一个配置文件中的值; 你可以在其中一个默认位置的 `application.properties` (或使用 `spring.config.name` 选择的任何其他基本名称) 中为应用程序提供默认值; 然后, 可以在运行时使用位于其中一个自定义位置的不同文件覆盖这些默认值
>如果使用环境变量而不是系统属性, 则大多数操作系统都不允许使用句点分隔的键名, 但你可以使用下划线 (例如, 使用 `SPRING_CONFIG_NAME` 而不是 `spring.config.name`)
>如果应用程序在容器中运行, 则可以使用 JNDI 属性 (在 `java:comp/env` 中) 或 servlet 上下文初始化参数来代替环境变量或系统属性

#### 特定于配置文件的属性
除 `application.properties` 文件外, 还可以使用以下命名约定定义特定于配置文件的属性: `application-{profile}.properties`; 环境具有一组默认配置文件 (默认情况下为 `[default]`), 如果未设置活动配置文件, 则使用这些配置文件; 换句话说, 如果未显式激活任何配置文件, 则会加载 `application-default.properties` 中的属性  
特定于配置文件的属性从与标准 `application.properties` 相同的位置加载, 特定于配置文件的文件始终覆盖非特定文件, 无论特定于配置文件的文件是在打包的 jar 内部还是外部  
如果指定了多个配置文件, 则应用最后获胜策略; 例如, `spring.profiles.active` 属性指定的配置文件是在通过 SpringApplication API 配置的配置文件之后添加的, 因此优先
>如果在 `spring.config.location` 中指定了任何文件, 则不考虑这些文件的特定于配置文件的变体; 如果要使用特定于配置文件的属性, 请使用 spring.config.location 中的目录

#### 属性中的占位符
`application.properties` 中的值在使用时通过现有环境进行过滤, 因此你可以返回先前定义的值 (例如, 从系统属性)
```
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```
>你还可以使用此技术创建现有Spring Boot属性的 "简短" 变体; 详见 [第 77.4 节 "使用 '短' 命令行参数"](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/howto-properties-and-configuration.html#howto-use-short-command-line-arguments) 操作方法

#### 加密属性
Spring Boot 没有为加密属性值提供任何内置支持, 但是它确实提供了修改 Spring 环境中包含的值所必需的钩子点; `EnvironmentPostProcessor` 接口允许你在应用程序启动之前操作 `Environment`; 详见 [第 76.3 节 "在开始之前自定义环境或 ApplicationContext"](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/howto-spring-boot-application.html#howto-customize-the-environment-or-application-context)  
如果你正在寻找一种存储凭据和密码的安全方法, [Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/) 项目支持在 [HashiCorp Vault](https://www.vaultproject.io/) 中存储外部化配置。

#### 使用 YAML 代替 Properties
[YAML](https://yaml.org/) 是 JSON 的超集, 因此是用于指定分层配置数据的便捷格式; 只要在类路径上有 [SnakeYAML](https://bitbucket.org/asomov/snakeyaml) 库, `SpringApplication` 类就会自动支持 YAML 作为 Properties 的替代
>如果使用 "starters", `spring-boot-starter` 会自动提供 `SnakeYAML`

##### 加载 YAML
Spring Framework 提供了两个方便的类可用于加载 YAML 文档; `YamlPropertiesFactoryBean` 将 YAML 加载为 `Properties`, `YamlMapFactoryBean` 将 YAML 加载为 `Map`  
例如, 请考虑以下 YAML 文档:
```
environments:
	dev:
		url: https://dev.example.com
		name: Developer Setup
	prod:
		url: https://another.example.com
		name: My Cool App
```
前面的示例将转换为以下属性:
```
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```
YAML 列表表示为带有 `[index]` 解除引用的属性键; 例如, 请考虑以下 YAML:
```
my:
servers:
	- dev.example.com
	- another.example.com
```
前面的示例将转换为这些属性:
```
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```
要使用 Spring Boot 的 `Binder` 工具类 (这是 `@ConfigurationProperties` 所做的) 绑定到这样的属性, 你需要在目标 bean 中有 `java.util.List (或 Set)` 类型的一个属性, 你需要提供一个 `setter` 或者用可变值初始化它; 例如, 以下示例绑定到前面显示的属性
```
@ConfigurationProperties(prefix="my")
public class Config {

	private List<String> servers = new ArrayList<String>();

	public List<String> getServers() {
		return this.servers;
	}
}
```
##### 在 Spring 环境中将 YAML 暴露为属性
`YamlPropertySourceLoader` 类可用于在 Spring 环境中将 YAML 暴露为 `PropertySource`; 这样做可以使用带占位符语法的 `@Value` 注解来访问 YAML 属性

##### 多档案的 YAML 文件
你可以使用 `spring.profiles` 键在单个文件中指定多个特定于配置文件的 YAML 文档, 以指示文档何时应用, 如以下所示
```
server:
	address: 192.168.1.100
---
spring:
	profiles: development
server:
	address: 127.0.0.1
---
spring:
	profiles: production & eu-central
server:
	address: 192.168.1.120
```
在以上示例中, 如果 `development` 配置文件处于活动状态, 则 `server.address` 属性为 `127.0.0.1`; 同样, 如果 `production` 和 `eu-central` 配置文件处于活动状态, 则 `server.address` 属性为 `192.168.1.120`; 如果未启用 `development`,  `production` 和 `eu-central` 配置文件, 则该属性的值为 `192.168.1.100`

>**参考:**  
[Externalized Configuration](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config)
