### 使用 JSR 330 标准注解
从 Spring 3.0 开始, Spring 提供对 JSR-330 标准注解 (依赖注入) 的支持; 这些注解的扫描方式与 Spring 注解相同; 你只需要在类路径中包含相关的 jar  
>如果你使用的是 Maven, 则 `javax.inject` 构件可在标准 Maven 存储库中找到 (https://repo1.maven.org/maven2/javax/inject/javax.inject/1/), 你可以将以下依赖项添加到文件 pom.xml
>```
><dependency>
>    <groupId>javax.inject</groupId>
>    <artifactId>javax.inject</artifactId>
>    <version>1</version>
></dependency>
>```

#### 使用 `@Inject` 和 `@Named` 依赖注入
TODO

####  `@Named` 和 `@ManagedBean`: 等同于 `@Component` 注解
TODO

#### JSR-330 标准注解的局限
TODO

>**参考:**  
[Using JSR 330 Standard Annotations](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-standard-annotations)
