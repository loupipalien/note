### 安装 Spring Boot
Spring Boot 可以与 "经典" Java 开发工具一起使用, 也可以作为命令行工具安装; 无论哪种方式, 你都需要 [Java SDK v1.8](https://www.java.com/) 即以上; 在你开始之前, 你可以使用以下命令行检查你当前安装的 Java:
```
$ java -version
```
如果你是 Java 开发新手或者你想尝试使用 Spring Boot, 你可能想首向尝试下 [Spring Boot CLI (Command Line Interface)](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#getting-started-installing-the-cli); 如若不是, 请继续阅读 "经典的" 安装说明

#### Java 开发者的安装说明
你可以像使用任何标准的 Java 库一样使用 Spring Boot; 像这样, 将合适的
`spring-boot-*.jar` 文件包含在你的 classpath 中; Spring Boot 并不需要任何特殊的工具集成, 所以你可以使用任意的 IDE 或者文本编辑器; 所以, Spring Boot 应用程序并没有任何特殊, 因此你可以像其他 Java 程序一样运行和调试 Spring Boot 应用程序  
尽管你可以拷贝 Spring Boot jars, 我们通常推荐你使用一个支持依赖管理的构建工具 (例如 Maven 或 Gradle)

##### Maven 安装
Spring Boot 兼容 Apache Maven 3.3 及以上; 如果你还没有安装 Maven, 你可以依照 [maven.apache.org] (https://maven.apache.org/)上的说明进行操作  
>在许多操作系统上, Maven 可以使用包管理器安装; 如果你使用 OSX Homebrew, 可以尝试 `brew install maven`; Ubuntu 用户可以运行 `sudo apt-get install maven`; Windows 用户可以使用 [Chocolatey](https://chocolatey.org/) 在命令行提示符 (管理员) 中运行 `choco install maven`

Spring Boot 依赖使用 `org.springframework.boot``groupId`; 一般的, 你的 Maven POM 文件继承 `spring-boot-starter-parent` 项目, 并且声明一个或多个 "[Starts](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-starter)" 依赖; Spring Boot 同时也提供一个可选的 [Maven 插件](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#build-tool-plugins-maven-plugin) 来创建可执行 jars  
以下列表展示了一个通用的 `pom.xml` 文件
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<!-- Inherit defaults from Spring Boot -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.3.RELEASE</version>
	</parent>

	<!-- Add typical dependencies for a web application -->
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>

	<!-- Package as an executable jar -->
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```
>`spring-boot-starter-parent` 是使用 Spring Boot 的好方法, 但它可能并不总是适合; 有时你可能需要继承一个不同的父 POM, 或者你可能不喜欢我们的默认设置; 这种情况, 使用 `import` 范围的另一种可选办法见 [13.2.2 小结 "没有父 POM 时使用 Spring Boot"](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-maven-without-a-parent)

##### Gradle 安装
TODO

#### 安装 Spring Boot CLI
Spring Boot CLI (Command Line Interface) 是一个命令行工具, 你可以使用它与 Spring 来快速生成原型; 它允许你运行 Groovy 脚本, 这意味着你有一个没有太多样板代码的类 Spring 语法  
你也可以不用 CLI 来使用 Spring Boot, 但它绝对是实现 Spring 应用程序的最快方法

##### 手动安装
你可以从 Spring 软件库下载 Spring CLI 发行版:
- [spring-boot-cli-2.1.3.RELEASE-bin.zip](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.1.3.RELEASE/spring-boot-cli-2.1.3.RELEASE-bin.zip)
- [spring-boot-cli-2.1.3.RELEASE-bin.tar.gz](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.1.3.RELEASE/spring-boot-cli-2.1.3.RELEASE-bin.tar.gz)

也提供罪行的 [快照发行版](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/)  
下载完成后, 按照解压包中的 [INSTALL.txt](https://raw.github.com/spring-projects/spring-boot/v2.1.3.RELEASE/spring-boot-project/spring-boot-cli/src/main/content/INSTALL.txt) 说明安装; 总之, 在 `zip` 文件中的 `bin/` 目录有一个 `spring` 脚本 (对于 WINDOWS 是 `spring.bat`); 另外, 你可以使用 `java -jar` 运行 `.jar` 文件 (这个脚本可以帮你确认 classpath 是否正确)

##### 使用 SDKMAN 安装
TODO

##### OSX Homebrew 安装
TODO

##### MacPorts 安装
TODO

##### 命令行补全
TODO

##### Windows Scoop 安装
TODO

##### Spring CLI 示例快速入门
你可以使用以下 web 应用程序来测试你的安装; 首先, 创建一个名为 `app.groovy` 的文件, 如下:
```
@RestController
class ThisWillActuallyRun {

	@RequestMapping("/")
	String home() {
		"Hello World!"
	}

}
```
在 shell 中运行如下命令
```
$ spring run app.groovy
```
>首次运行你的应用程序是很慢的, 因为要下载依赖, 后续运行是非常快的

在你的浏览器中打开 `localhost:8080`, 你可以看到如下输出
```
Hello World!
```

#### 从早期的 Spring Boot 版本升级
TODO

>**参考:**
[Installing Spring Boot](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#getting-started-installing-spring-boot)
