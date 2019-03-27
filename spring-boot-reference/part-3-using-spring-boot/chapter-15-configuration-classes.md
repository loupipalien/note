### 配置类
你无需将 `@Configuration` 放在所有类上; `@Import` 注解可用于导入其他配置类; 或者你可以使用 `@ComponentScan` 自动获取所有 Spring 组件, 包括 `@Configuration` 类
>许多使用 XML 配置的 Spring 示例已在 Internet 上发布; 如果可能, 请始终尝试使用等效的基于 Java 的配置; 搜索 `Enable*` 注解是一个很好的开始

#### 导入 XML 配置
如果你必须使用基于 XML 的配置，我们建议你仍然使用 `@Configuration`类, 然后你可以使用 `@ImportResource` 注解来加载 XML 配置文件

>**参考:**
[Configuration Classes](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-configuration-classes)
