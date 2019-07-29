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
将你的项目配置为从 `spring-boot-starter-parent` 继承, 设置 `parent` 如下
```
<!-- Inherit defaults from Spring Boot -->
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.1.3.RELEASE</version>
</parent>
```
>你只需要在此依赖项上指定 Spring Boot 的版本号, 如果导入其他启动器, 则可以放心的省略版本号


你还可以通过覆盖项目中的属性来覆盖单个依赖; 例如, 要升级到另一个 Spring Data 版本, 你可以添加如下配置到你的 `pom.xml` 中
```
<properties>
	<spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```
>查看 `spring-boot-dependencies` [pom](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml) 中支持的属性列表

##### 在没有父 POM 的情况下使用 Spring Boot
不是每个人都喜欢继承 `spring-boot-starter-parent` POM; 你可能拥有自己需要使用的公司标准父级, 或者你可能更愿意明确声明所有的 Maven 配置  
如果你不想使用 `spring-boot-starter-parent`, 你仍然可以通过使用 `scope = import` 依赖来保留依赖管理 (但不是插件管理) 的好处, 如下所示:
```
<dependencyManagement>
		<dependencies>
		<dependency>
			<!-- Import dependency management from Spring Boot -->
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>2.1.3.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```
如上所述, 以上的示例设置不允许你使用属性覆盖依赖的版本; 要达到同样的目的, 你需要在项目的 `dependencyManagement` 中的 `spring-boot-dependencies` 条目前添加一个条目; 例如, 要升级到另一个 Spring Data 版本, 可以将以下元素添加到 `pom.xml` 中
```
<dependencyManagement>
	<dependencies>
		<!-- Override Spring Data release train provided by Spring Boot -->
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-releasetrain</artifactId>
			<version>Fowler-SR2</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>2.1.3.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```
>在前面的示例中, 我们指定了一个 BOM, 但是可以以相同的方式覆盖任何依赖

##### 使用 Spring Boot Maven 插件
Spring Boot 包含一个可以把项目打包成一个可执行 jar 的 [Mavin Plugin](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#build-tool-plugins-maven-plugin); 如果你想使用它, 可添加插件到 `<plugins>` 段中, 如下例所示
```
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```
>如果使用 spring-boot-starter-parent 作为父 POM, 则只需添加插件; 除非你要更改父级中定义的设置, 否则无需对其进行配置

#### Gradle
TODO

#### Ant
TODO

#### Starters
启动器是一组方便的依赖关系描述符, 你可以在应用程序中包含它们; 你可以获得所需的所有 Spring 和相关技术的一站式服务, 而无需查看示例代码并且复制粘贴其依赖描述符; 例如, 如果要开始使用 Spring 和 JPA 进行数据库访问, 请在项目中包含 `spring-boot-starter-data-jpa` 依赖  
启动器包含许多依赖, 这些依赖是使项目快速启动和运行所需的依赖, 以及一组受支持的托管传递性依赖
```
关于名称
所有官方首发都遵循类似的命名模式; `spring-boot-starter-*`, 其中 * 是特定类型的应用程序; 此命名结构旨在帮助你找到启动器; 许多 IDE 中的 Maven 集成允许你按名称搜索依赖; 例如, 安装了适当的 Eclipse 或 STS 插件后, 你可以在 POM 编辑器中按 ctrl-space 并输入 "spring-boot-starter" 以获取完整列表
正如 "[创建自己的启动器](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-custom-starter)" 部分所述, 第三方启动器不应该以 `spring-boot` 开头, 因为它是为官方 Spring Boot 构件保留的; 第三方启动器通常以项目名称开头; 例如, 名为 `thirdpartyproject` 的第三方启动项目通常被命名为 `thirdpartyproject-spring-boot-starter`
```
以下是 Spring Boot 提供的 `org.springframework.boot` 组下的应用程序启动器

| 名称 | 描述 |
| :--- | :--- |
| spring-boot-starter | 核心启动器, 包括自动配置支持, 日志记录和 YAML |
| spring-boot-starter-activemq | 使用 Apache ActiveMQ 进行 JMS 消息传递的启动器 |
| spring-boot-starter-ampq | 使用 Spring AMQP 和 Rabbit M Q的启动器 |
| spring-boot-starter-aop | 使用 Spring AOP 和 AspectJ 进行切面编程的启动器 |
| spring-boot-starter-artemis | 使用  Apache Artemis 进行 JMS 消息传递的启动器 |
| spring-boot-starter-batch | 使用 Spring Batch 的启动器 |
| spring-boot-starter-cache | 使用 Spring Framework 支持缓存的启动器 |
| spring-boot-starter-cloud-connectors | 使用 Spring Cloud Connectors 的启动器, 简化了 Cloud Foundry 和 Heroku 等云平台的服务连接 |
| spring-boot-starter-data-cassandra | 使用 Cassandra 分布式数据库和 Spring Data Cassandra 的启动器 |
| spring-boot-starter-data-cassandra-reactive | 使用 Cassandra 分布式数据库和 Spring Data Cassandra Reactive 的启动器 |
| spring-boot-starter-data-couchbase | 使用 Couchbase 面向文档的数据库和 Spring Data Couchbase 的启动器 |
| spring-boot-starter-data-couchbase-reactive | 使用 Couchbase 面向文档的数据库和 Spring Data Couchbase Reactive 的启动器 |
| spring-boot-starter-data-elasticsearch | 使用 Elasticsearch 搜索和分析引擎以及 Spring Data Elasticsearch 的启动器 |
| spring-boot-starter-data-jdbc | 使用 Spring Data JDBC 的启动器 |
| spring-boot-starter-data-jpa | 使用 Spring Data JPA 和 Hibernate 的启动器 |
| spring-boot-starter-data-ldap | 使用 Spring Data LDAP 的启动器 |
| spring-boot-starter-data-mongodb | 使用 MongoDB 面向文档的数据库和 Spring Data MongoDB 的启动器 |
| spring-boot-starter-data-mongodb-reactive | 使用 MongoDB 面向文档的数据库和 Spring Data MongoDB Reactive 的启动器 |
| spring-boot-starter-data-neo4j | 使用 Neo4j 图形数据库和 Spring Data Neo4j 的启动器 |
| spring-boot-starter-data-redis | 在 Spring Data Redis 和 Lettuce 客户端上使用 Redis 键值数据存储的启动器 |
| spring-boot-starter-data-redis-reactive | 在 Spring Data Redis Reactive 和 Lettuce 客户端上使用 Redis 键值数据存储的启动器 |
| spring-boot-starter-data-rest | 使用 Spring Data REST 通过 REST 暴露 Spring Data 库的启动器 |
| spring-boot-starter-data-solr | 在 Spring Data Solr 中使用 Apache Solr 搜索平台的启动器 |
| spring-boot-starter-freemarker | 使用 FreeMarker 视图构建 MVC Web 应用程序的启动器 |
| spring-boot-starter-groovy-templates | 使用 Groovy Templates 视图构建 MVC Web 应用程序的启动器 |
| spring-boot-starter-hateoas | 使用 Spring MVC 和 Spring HATEOAS 构建基于超媒体的 RESTful Web 应用程序的启动器 |
| spring-boot-starter-integration | 使用 Spring 集成的启动器 |
| spring-boot-starter-jdbc | 将 JDBC 与 HikariCP 连接池一起使用的启动器 |
| spring-boot-starter-jersey | 使用 JAX-RS 和 Jersey 构建 RESTful Web 应用程序的启动器; spring-boot-starter-web 的替代品 |
| spring-boot-starter-jooq | 使用 jOOQ 访问 SQL 数据库的启动器; spring-boot-starter-data-jpa 或 spring-boot-starter-jdbc 的替代品 |
| spring-boot-starter-json | 读写 json 的启动器 |
| spring-boot-starter-jta-atomikos | 使用 Atomikos 的 JTA 事务的启动器 |
| spring-boot-starter-jta-bitronix | 使用 Bitronix 的 JTA 事务的 |
| spring-boot-starter-mail | 使用 Java Mail 和 Spring Framework 的电子邮件发送支持的启动器 |
| spring-boot-starter-mustache | 使用 Mustache 视图构建 Web 应用程序的启动器 |
| spring-boot-starter-oauth2-client | 使用 Spring Security 的 OAuth2/OpenID Connect 客户端功能的启动器 |
| spring-boot-starter-oauth2-resource-server | 使用 Spring Security 的 OAuth2 资源服务器功能的启动器 |
| spring-boot-starter-quartz | 使用 Quartz 调度器的启动器 |
| spring-boot-starter-security | 使用 Spring Security 的启动器 |
| spring-boot-starter-test | 使用 JUnit, Hamcrest 和 Mockito 等库来测试 Spring Boot 应用程序的启动器 |
| spring-boot-starter-thymeleaf | 使用 Thymeleaf 视图构建 MVC Web 应用程序的启动器 |
| spring-boot-starter-validation | 使用 Java Bean Validation 和 Hibernate Validator 的启动器 |
| spring-boot-starter-web | 使用 Spring MVC 构建 Web (包括RESTful) 应用程序的启动器; 使用 Tomcat 作为默认嵌入式容器 |
| spring-boot-starter-web-services | 使用 Spring Web Services 的启动器 |
| spring-boot-starter-webflux | 使用 Spring Framework Reactive Web 支持的构建 WebFlux 应用程序的启动器 |
| spring-boot-starter-websocket | 使用 Spring Framework WebSocket 支持的构建 WebSocket 应用程序的启动器 |

除应用程序启动器外, 还可以使用以下启动器添加 [生产级别](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#production-ready) 特性

| 名称 | 描述 |
| :--- | :--- |
| spring-boot-starter-actuator | 使用 Spring Boot 的 Actuator 的启动器，它提供生产级别特性, 帮助你监控和管理你的应用程序 |

最后, 如果要排除或替换特定的技术面, Spring Boot 还包括以下启动器

| 名称 | 描述 |
| :--- | :--- |
| spring-boot-starter-jetty | 使用 Jetty 作为嵌入式 servlet 容器启动器; spring-boot-starter-tomcat 的替代品 |
| spring-boot-starter-log4j2 | 使用Log4j2进行日志记录的启动器; spring-boot-starter-logging 的替代品 |
| spring-boot-starter-logging | 使用 Logback 进行日志记录启动器; 默认日志启动器 |
| spring-boot-starter-reactor-netty | 使用 Reactor Netty 作为嵌入式响应式 HTTP 服务器的启动器 |
| spring-boot-starter-tomcat | 使用 Tomcat 作为嵌入式 servlet 容器的启动器; spring-boot-starter-web 使用的默认 servlet 容器启动器 |
| spring-boot-starter-undertow | 使用 Undertow 作为嵌入式 servlet 容器的启动器; spring-boot-starter-tomcat 的替代品 |

>有关其他社区贡献启动器的列表, 请参阅 GitHub 上 `spring-boot-starters` 模块中的 [README 文件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/README.adoc)

>**参考:**
[Build Systems](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-build-systems)
