### 部署你的第一个 Spring Boot 应用
本节介绍如何开发一个简单的 "Hello World!" Web 应用程序, 该应用程序使用了 Spring Boot 的一些主要功能; 我们使用 Maven 来构建这个项目, 因为大多数 IDE 都支持它
> [spring.io](https://spring.io/) 网站包含许多使用 Spring Boot 的 "入门" 指南; 如果你需要解决特定问题, 请先从这里查找
你可以直接访问 [start.spring.io](https://start.spring.io/) 并从依赖关系搜索器中选择 "Web" 来快速执行以下步骤; 这样做会生成一个新的项目结构, 以便你可以立即开始编码; 有关更多详细信息请查看 Spring Initializer 文档

在开始之前, 打开一个终端并运行以下命令以确认你安装了的 Java 和 Maven 的有效版本
```
$ java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```
```
$ mvn -v
Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-17T14:33:14-04:00)
Maven home: /usr/local/Cellar/maven/3.3.9/libexec
Java version: 1.8.0_102, vendor: Oracle Corporation
```
>此示例需要在它自己的文件夹中创建; 后续说明假定你已创建了一个合适的文件夹, 并且它是你当前的目录

#### 创建一个 POM
我们首先创建一个 Maven 的 `pom.xml` 文件; `pom.xml` 是用于构建项目的配置; 打开你喜欢的文本编辑器并添加以下内容:
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.3.RELEASE</version>
	</parent>

	<!-- Additional lines to be added here... -->

</project>
```
上面的清单应该为你提供有效的构建; 你可以通过运行 `mvn package` 来测试 (现在, 你可以忽略 "jar 将为空 - 没有内容被标记为包含!" 的警告)
>此时, 你可以将项目导入 IDE (大多数现代 Java IDE 包括了对 Maven 的内置支持); 为简单起见, 我们继续为此示例使用纯文本编辑器

#### 添加 ClassPath 依赖
Spring Boot 提供了许多 "Starters", 可以让你将 jar 添加到类路径中; 我们的示例应用程序已经在 POM 的 `parent` 段中添加了 `spring-boot-starter-parent`; `spring-boot-starter-parent` 是一个特殊的启动器, 提供了有用的 `Maven` 默认; 它还提供了一个 `ependency-management` 段, 以便你可以省略依赖项的 `version` 标记  
其他 "Starters" 提供了在开发特定类型的应用程序时可能需要的依赖; 由于我们开发 Web 应用程序, 因此我们添加了一个 `spring-boot-starter-web` 依赖; 在此之前, 我们可以通过运行以下命令来查看当前的依赖
```
$ mvn dependency:tree

[INFO] com.example:myproject:jar:0.0.1-SNAPSHOT
```
`mvn dependency：tree` 命令打印了你的项目依赖树; 你可以看到 `spring-boot-starter-parent` 本身不提供依赖; 要添加必要的依赖项, 请编辑 `pom.xml` 文件, 并在 `parent` 段下面添加 `spring-boot-starter-web` 依赖
```
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
```
如果你再次运行 `mvn dependency:tree`, 你可以现在看到有大量的依赖, 包括 Tomcat web 服务器和 Spring Boot

#### 编写代码
为了完成我们的应用, 我们需要创建一个 Java 文件; 默认的, Maven 编译源文件在 `src/main/java`, 所以你需要创建该文件夹结构, 然后添加一个名为 `src/main/java/Example.java` 的文件, 包含以下代码
```
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

	@RequestMapping("/")
	String home() {
		return "Hello World!";
	}

	public static void main(String[] args) {
		SpringApplication.run(Example.class, args);
	}

}
```
虽然这里的代码不多, 但后续还有很多代码; 我们接下来将逐步介绍重要的章节

##### @RestController 和 @RequestMapping 注解
我们的 `Example` 类的第一个注解是 `@RestController`; 这是一个原型注释; 它为阅读代码的人提供了提示, 并且为 Spring 提供这个类是一个特定角色的提示; 此时, 我们的类是一个 Web `@Controller`, 所以 Spring 在处理传入的 Web 请求时会考虑它  
`@RequestMapping` 注解提供 "路由" 信息; 它告诉 Spring 任何带有 `/` 路径的 HTTP 请求都应该映射到 `home` 方法; `@RestController` 注解告诉 Spring 将结果字符串直接返回调用者
>@RestController 和 @RequestMapping 注解 是 Spring MVC 的注解 (它们不是 Spring Boot 特有的); 详见 Spring 参考文档 [MVC 章节](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/web.html#mvc)

##### @EnableAutoConfiguration 注解
第二个类级别注解是 `@EnableAutoConfiguration`; 这个注解告诉 Spring Boot 根据你添加的 jar 依赖关系去 "猜测" 你想要如何配置; 由于 `spring-boot-starter-web` 添加了 Tomcat 和 Spring MVC, 因此自动配置假定你正在开发 Web 应用程序并相应地设置 Spring
```
启动器和自动配置
自动配置旨在与 "启动器" 配合使用, 但这两个概念并不直接相关; 你可以在 "启动器" 之外自由选择 jar 依赖; Spring Boot 仍然尽力自动配置你的应用程序
```

##### "main" 方法
我们的应用程序的最后一部分是 `main` 方法; 这只是遵循 Java 约定的应用程序入口点的标准方法; 我们的 `main` 方法通过调用 `run` 来委托 Spring Boot 的 `SpringApplication`类; `SpringApplication` 引导我们的应用程序启动 Spring, 然后启动自动配置的 Tomcat Web 服务器; 我们需要将 `Example.class` 作为参数传递给 `run` 方法以告诉 Spring 组件组件 `SpringApplication`; `args` 数组也可以传递任何命令行参数

#### 运行示例
此时, 你的应用程序应该可以工作; 由于你使用了 `spring-boot-starter-parent` POM, 因此你可以使用一个有用的 `run` 目标来启动应用程序; 在项目根目录下键入 `mvn spring-boot:run` 以启动应用程序; 你应该看到类似于以下内容的输出
```
$ mvn spring-boot:run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.1.3.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.222 seconds (JVM running for 6.514)
```
如果你在 Web 浏览器打开 `localhost:8080`, 应该看到以下输出
```
Hello World!
```
优雅的推出应用程序请按 `ctrl-c`

#### 创建一个可执行 jar
我们通过创建一个可在生产运行的完全自包含的可执行 jar 文件来完成示例; 可执行 jar (有时称为 "fat jar") 是包含已编译类以及代码需要运行的所有 jar 依赖的归档
```
可执行 jar 和 Java
Java 没有提供加载嵌套 jar 文件的标准方法 (jar 文件本身包含在 jar 中); 如果你要分发自包含的应用程序, 这可能会有问题
为了解决这个问题, 许多开发人员使用 "uber" jars; uber jar 将应用程序所有依赖中的所有类打包到一个存档中; 这种方法的问题在于很难看出你的应用程序中有哪些库; 如果在多个 jar 中有相同的文件名 (但具有不同的内容), 也可能会有问题
Spring Boot 采用 [不同的方法](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#executable-jar), 让你直接嵌套 jar
```
为了创建可执行 jar, 我们需要将 `spring-boot-maven-plugin` 添加到 `pom.xml`中; 为此请在依赖部分下方插入以下行
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
> `spring-boot-starter-parent` POM 包含 `<executions>` 配置以绑定重新打包目标; 如果你不使用父 POM, 则需要自己声明此配置; 详见 [插件文档](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/maven-plugin/usage.html)

保存你的 `pom.xml` 文件并在命令行中运行 `mvn package`, 如下所示
```
$ mvn package

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.1.3.RELEASE:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```
如果查看 `target` 目录, 你应该可以看到 `myproject-0.0.1-SNAPSHOT.jar`; 这个文件大概 10MB 大小; 如果你想查看内容, 可以使用 `jar tv`, 如下所示
```
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```
你应该在目标目录中可以看到名为小得多的 `myproject-0.0.1-SNAPSHOT.jar.original` 的文件; 这是 Maven在 Spring Boot 重新打包之前创建的原始 jar 文件  
要运行该应用程序, 请使用 `java -jar`命令, 如下所示
```
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.1.3.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.536 seconds (JVM running for 2.864)
```
类似之前, 要退出应用程序, 请按 `ctrl-c`

>**参考:**
[Developing Your First Spring Boot Application](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#getting-started-first-application)
