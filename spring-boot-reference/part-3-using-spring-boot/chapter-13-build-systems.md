### 构建系统
强烈推荐你选择支持 [依赖管理](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-dependency-management) 的构建系统, 它可以从 "Maven Central" 仓库中拉去公开的构件; 我们推荐你使用 Maven 或者 Gradle; Spring Boot 使用其他构建系统也是有可能的 (例如 Ant), 但是没有非常好的支持

#### 依赖管理
每个版本的 Spring Boot 都提供了它支持的依赖的列表; 实际上, 你不需要为构建配置中的任何这些依赖提供版本, 因为 Spring Boot 会管理这些依赖, 当升级 Spring Boot 时, 这些依赖也会以一致的方式升级
>如果需要, 你仍然可以指定版本覆盖 Spring Boot 的推荐

列表包含可以与 Spring Boot 一起使用的所有 spring 模块以及第三方库列表; 该列表可作为标准的 [ills of Materials (spring-boot-dependencies)](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-maven-without-a-parent) 提供, 可与 Maven 或 Gradle 一起使用
>每个版本的 Spring Boot 都与 Spring Framework 的一个基本版本相关联; 我们强烈建议你不要指定其版本

#### Maven
Maven 用户可以继承 `spring-boot-starter-parent` 项目以获得合理的默认值; 父项目提供以下特性
- Java 1.8 作为默认的编译级别
- UTF-8 编码
- 继承自 `spring-boot-dependencies` POM 的 [依赖管理部分](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-dependency-management, 用于管理公共依赖的版本; 此依赖管理允许你在自己的 POM 中使用时省略这些依赖的 `<version>` 标签
- 使用 `repackage` 执行 id 执行的 [`repackage` 目标](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/maven-plugin/repackage-mojo.html)
- 合理的 [资源过滤](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html)
- 合理的插件配置 ([exec plugin](http://www.mojohaus.org/exec-maven-plugin/), [Git commit ID](https://github.com/ktoso/maven-git-commit-id-plugin), [shade](https://maven.apache.org/plugins/maven-shade-plugin/))
- 对于 `application.properties` 和 `application.yml` 合理的资源过滤, 包括特定剖面的配置文件 (例如, application-dev.properties 和 application-dev.yml)

注意, 由于 `application.properties` 和 `application.yml` 文件接受 Spring 样式占位符 (`$ {...}`), 因此 Maven 过滤更改为使用 `@..@` 占位符; (你可以通过设置名为 `resource.delimiter`的 Maven 属性来覆盖它)

##### 继承 Starter Parent




>**参考:**
[Build Systems](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-build-systems)
