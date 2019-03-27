### 使用 @SpringBootApplication 注解
许多 Spring Boot 开发人员喜欢他们的应用程序使用自动配置, 组件扫描, 并能够在他们的 "应用程序类" 上定义额外的配置; 单个 `@SpringBootApplication` 注解可用于启用这三个功能, 它们是：
- `@EnableAutoConfiguration`: 开启 [Spring Boot 的自动配置机制](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)
- `@ComponentScan`: 在应用程序所在的包上启用 `@Component` 扫描 (见 [最佳实践](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-structuring-your-code))
- `@Configuration`: 允许在上下文中注册额外的 bean 或导入其他配置类

`@SpringBootApplication` 注解等同于使用 `@Configuration`, `@EnableAutoConfiguration` 和 `@ComponentScan` 及其默认属性, 如以下所示:
```
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```
>`@SpringBootApplication` 还提供别名来自定义 `@EnableAutoConfiguration` 和 `@ComponentScan` 的属性

>这些功能都不是必需的, 你可以选择通过它启用的任何功能替换此单个注释; 例如, 你可能不希望在应用程序中使用组件扫描

```
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.ComponentScan
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@EnableAutoConfiguration
@Import({ MyConfig.class, MyAnotherConfig.class })
public class Application {

	public static void main(String[] args) {
			SpringApplication.run(Application.class, args);
	}

}
```
>在此示例中, `Application` 与任何其他 Spring Boot 应用程序一样, 只是未自动检测 `@Component-annotated` 类, 并且显式导入用户定义的 bean (见 `@Import`)

>**参考:**
[Using the @SpringBootApplication Annotation](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-using-springbootapplication-annotation)
