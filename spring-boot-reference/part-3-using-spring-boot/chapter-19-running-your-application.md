### 运行你的应用程序
将应用程序打包为 jar 并使用嵌入式 HTTP 服务器的最大优势之一是, 你可以像运行任何其他服务器一样运行应用程序; 调试 Spring Boot 应用程序也很容易, 你不需要任何特殊的 IDE 插件或扩展
>本节仅介绍基于 jar 的打包, 如果你选择将应用程序打包为 war 文件, 则应参阅服务器和 IDE 文档

#### 在 IDE 中运行
你可以在 IDE 将 Spring Boot 应用程序作为简单的Java应用程序运行; 但你首先需要导入项目; 导入步骤因 IDE 和构建系统而异; 大多数 IDE 可以直接导入 Maven 项目; 例如, Eclipse 用户可以从 "File" 菜单中选择 "Import..." -> "Existing Maven Projects"  
如果无法将项目直接导入 IDE, 则可以使用构建插件生成 IDE 元数据; Maven 包含 [Eclipse](https://maven.apache.org/plugins/maven-eclipse-plugin/) 和 [IDEA](https://maven.apache.org/plugins/maven-idea-plugin/) 的插件, Gradle 提供 [各种 IDE](https://docs.gradle.org/4.2.1/userguide/userguide.html) 的插件
>如果你不小心将 Web 应用程序运行了两次, 则会看到 "端口已在使用中" 错误; STS 用户可以使用 "重新启动" 按钮而不是 "运行" 按钮来确保关闭任何现有实例

#### 作为打包应用程序运行
如果使用 Spring Boot Maven 或 Gradle 插件创建可执行 jar, 则可以使用 `java -jar`运行应用程序, 如下所示
```
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```
也可以运行启用了远程调试支持的打包应用程序; 这样做可以将调试器附加到打包的应用程序, 如下所示
```
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

#### 使用 Maven 插件
Spring Boot Maven 插件包含一个可用于快速编译和运行应用程序的运行目标; 应用程序以分解形式运行, 就像在 IDE 中一样; 以下示例展示了运行 Spring Boot 应用程序的典型 Maven 命令
```
$ mvn spring-boot:run
```
你可能还想使用 `MAVEN_OPTS` 的操作系统环境变量, 如下所示
```
$ export MAVEN_OPTS=-Xmx1024m
```

#### 使用 Gradle 插件
TODO

#### 热插拔
由于 Spring Boot 应用程序只是普通的 Java 应用程序, 因此 JVM 热插拔应该是开箱即用的; JVM 热插拔在某种程度上受限于它可以替换的字节码; 要获得更完整的解决方案, 可以使用 [JRebel](https://zeroturnaround.com/software/jrebel/)  
`spring-boot-devtools` 模块还支持快速重启应用程序; 有关详细信息, 见本章后面的 [第20章 "开发人员工具"](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-devtools) 章节和 [热插拔 "操作方法"](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#howto-hotswapping)

>**参考:**
[Running Your Application](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-running-your-application)
