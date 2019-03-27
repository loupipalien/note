### 自动导入
Spring Boot 自动配置会尝试根据你添加的 jar 依赖自动配置 Spring 应用程序; 例如, 如果 HSQLDB 在你的类路径上, 并且你尚未手动配置任何数据库连接 bean, 则 Spring Boot 会自动配置一个内存数据库  
你需要通过将 `@EnableAutoConfiguration` 或 `@SpringBootApplication` z注解添加到其中一个 `@Configuration` 注解的类上开启自动配置
>你应该只添加 `@SpringBootApplication` 或 `@EnableAutoConfiguration` 注解; 我们通常建议你仅将其中一个注解添加到 `@Configuration` 注解的类上

#### 逐步替换自动配置
自动配置是无侵入的; 在任何时候, 你都可以定义你自己的配置来替换特定部分的自动配置; 例如, 如果你添加一个你自己的 `DataSource` bean, 会丢弃默认的嵌入式数据库支持
如果你需要了解当前正在应用的自动配置以及原因, 请使用 `--debug` 开关启动你的应用程序; 这样做可以为已选择的核心记录器启用调试日志, 并将条件报告记录到控制台

#### 禁用特定的自动配置类
如果发现正在应用你不需要的特定自动配置类, 则可以使用 `@EnableAutoConfiguration` 的 `exclude` 属性禁用它们, 如以下示例所示
```
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```
如果类不在类路径中, 则可以使用注解的 `excludeName` 属性, 并指定全限定名; 最后, 你还可以使用 `spring.autoconfigure.exclude` 属性控制要排除的自动配置类列表
>您可以在注解级别和使用属性定义排除项

>**参考:**
[Auto-configuration](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)
