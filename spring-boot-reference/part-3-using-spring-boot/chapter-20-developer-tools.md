### 开发者工具
Spring Boot 包含一套额外的工具, 可以使应用程序开发体验更加愉快; `spring-boot-devtools` 模块可以包含在任何项目中, 以提供额外的开发功能; 为了包含 devtools 支持, 请将模块依赖添加到你的构建中, 如以下 Maven 和 Gradle 清单中所示
###### Maven
```
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-devtools</artifactId>
		<optional>true</optional>
	</dependency>
</dependencies>
```
###### Gradle
```
configurations {
	developmentOnly
	runtimeClasspath {
		extendsFrom developmentOnly
	}
}
dependencies {
	developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

>运行打包应用程序时会自动禁用开发者工具; 如果你的应用程序是使用 `java -jar` 启动的, 或者它是从特殊的类加载器启动的, 那么它将被视为 "生产应用程序"; 将依赖标记为 Maven 中的可选项或在 Gradle 中使用自定义 "仅供开发" 配置 (如上所示), 这是防止 devtools 被传递应用于使用你的项目的其他模块的最佳实践
>重打包的归档默认情况下不包含 devtools; 如果要使用 [某个远程 devtools 功能](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-devtools-remote), 则需要禁用 `excludeDevtools` 构建属性以包含它; Maven 和 Gradle 插件都支持该属性

#### 属性默认值
Spring Boot 支持的几个库使用缓存来提高性能; 例如, [模板引擎](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-template-engines) 缓存已编译的模板来避免重复解析模板文件; 此外, Spring MVC 可以在提供静态资源时为响应添加 HTTP 缓存请求头  
虽然缓存在生产中非常有用, 但在开发过程中可能会适得其反, 使你无法看到刚刚在应用程序中进行的更改; 因此, `spring-boot-devtools` 默认禁用缓存选项
缓存选项通常在 `application.properties` 文件的配置。例如, Thymeleaf 提供了 `spring.thymeleaf.cache` 属性; 这些属性不需要手动设置,  `spring-boot-devtools` 模块会自动应用合理的开发配置  
因为在开发 `Spring MVC` 和 `Spring WebFlux` 应用程序时, 你可能需要更多有关 Web 请求的信息, 所以开发者工具将为 `Web` 日志记录组启用 DEBUG 日志; 处理程序处理它时这将为你提供有关传入请求, 响应结果等的信息; 如果你希望记录所有请求详细信息 (包括可能的敏感信息), 你可以打开 `spring.http.log-request-details` 配置属性
>如果你不希望应用属性默认值, 则可以在 `application.properties` 中将 `spring.devtools.add-properties` 设置为 `false`
>开发者工具应用的完整属性列表, 见 [DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/tree/v2.1.3.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)

#### 自动重启
使用 `spring-boot-devtools` 的应用程序会在类路径上的文件发生更改时自动重启;  使用 IDE 工作时, 这是一个有用的特性, 因为它为代码更改提供了非常快速的回路反馈; 默认情况下, 将监视类路径上指向文件夹的任何条目的更改; 请注意, 某些资源 (如静态资源和视图模板) [无需重新启动应用程序](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-devtools-restart-exclude)
```
触发一次重启
由于 DevTools 监视类路径资源, 因此触发重新启动的唯一方法是更新类路径; 导致更新类路径的方式取决于你使用的 IDE; 在 Eclipse 中, 保存修改后的文件会导致更新类路径并触发重新启动; 在 IntelliJ IDEA 中, 构建项目 (`Build -> Build Project`) 具有相同的效果
```
>只要启用了分支, 你也可以使用受支持的构建插件 (Maven 和 Gradle) 启动应用程序, 因为 DevTools 需要一个独立的应用程序类加载器才能正常运行; 默认情况下, Gradle和 Maven 在类路径上检测到 DevTools 时会这样做
>与 LiveReload 一起使用时, 自动重启非常有效; 详见 [LiveReload 章节](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-devtools-livereload); 如果使用 JRebel, 则禁用自动重新启动以支持动态类重新加载; 其他 devtools 功能 (例如 LiveReload 和属性覆盖) 仍然可以使用
>DevTools 依赖于应用程序上下文的关闭钩子在重启期间来关闭它; 如果禁用了关闭钩子, 则无法正常工作 (`SpringApplication.setRegisterShutdownHook(false)`)
>当类路径上的条目更改, 在确定该更改应该触发重新启动时, DevTools会自动忽略名为的 `spring-boot`, `spring-boot-devtools`, `spring-boot-autoconfigure`, `spring-boot-actuator`, `spring-boot-starter` 项目
>DevTools 需要自定义 ApplicationContext 使用的 ResourceLoader; 如果你的应用程序已经提供了一个, 它将被进一步包装; 不支持在 ApplicationContext 上直接覆盖 getResource 方法
```
重新启动 vs 重新加载  
Spring Boot 提供的重启技术使用两个类加载器; 没有更改的类 (例如, 来自第三方 jar 的类) 将加载到 base 加载器中; 你正在开发的类将加载到 restart 类加载器中; 重新启动应用程序时, 将 restart 类加载器丢弃并创建一个新的类加载器; 这种方法意味着应用程序重新启动通常比 "冷启动" 快得多, 因为 base 加载器已经可填充并可用  
如果你发现重新启动对于您的应用程序来说不够快, 或者遇到类加载问题, 你可以考虑 `ZeroTurnaround 的 JRebel 重新加载技术; 这些通过在加载类时重写类使得它们更加适合用于重新加载
```

##### 记录在条件评估中更改
默认情况下, 每次应用程序重新启动时, 都会打印一个显示条件评估增量的报告; 当你做了添加或移除 beans 或者设置配置属性的更改, 该报告会显示应用程序自动配置的更改
要禁用报告的日志记录, 请设置以下属性
```
spring.devtools.restart.log-condition-evaluation-delta=false
```

##### 排除资源
某些资源在更改时不一定需要触发重启; 例如, 可以就地编辑 Thymeleaf 模板; 默认情况下, 更改 `/META-INF/maven`, `/META-INF/resources`, `/resources`, `/static/ public` 或 `/templates` 中的资源不会触发重新启动, 但会触发 [实重新加载](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-devtools-livereload); 如果要自定义这些排除项, 可以使用 `spring.devtools.restart.exclude` 属性; 例如, 仅排除 `/static` 和 `/public`, 你需要设置以下属性
```
spring.devtools.restart.exclude=static/**,public/**
```
>如果要保留这些默认值并添加其他排除项, 请使用 `spring.devtools.restart.additional-exclude` 属性

##### 监控其他路径
当你对不在类路径中的文件进行更改时, 你可能希望重新启动或重新加载应用程序; 为此, 请使用 `spring.devtools.restart.additional-paths` 属性配置其他路径以监视更改; 你可以使用前面描述的 `spring.devtools.restart.exclude` 属性来控制其他路径下的更改是触发完全重新启动还是 [实重新加载](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-devtools-livereload)

##### 禁止重启
如果你不想使用重新启动功能, 可以使用 `spring.devtools.restart.enabled` 属性将其禁用; 在大多数情况下, 你可以在 `application.properties` 中设置此属性 (这样做仍会初始化重新启动的类加载器, 因为它不会监视文件更改)  
如果需要完全禁用重新启动支持 (例如, 因为它不能与特定库一起使用), 则需要在调用 `SpringApplication.run(..)` 之前将 `spring.devtools.restart.enabled` 属性在 `System` 中设置为 `false`, 如下所示:
```
public static void main(String[] args) {
	System.setProperty("spring.devtools.restart.enabled", "false");
	SpringApplication.run(MyApp.class, args);
}
```

##### 使用触发器文件
如果使用 IDE 不断编译已更改文件的, 则可能更喜欢仅在特定时间触发重新启动; 为此, 你可以使用 "触发器文件", 这是一个特殊文件, 当你想要实际触发重新启动检查时, 必须对其进行修改; 更改文件只会触发检查, 只有在 Devtools 检测到执行了某些操作时才会重新启动; 触发器文件可以手动更新, 也可以使用 IDE 插件更新  
要使用触发器文件, 请将 `spring.devtools.restart.trigger-file` 属性设置为触发器文件的路径
>你可能希望将 `spring.devtools.restart.trigger-file`设置为全局配置, 以便所有项目有相同的行为方式

##### 自定义重启类加载器
如前面在 [Restart vs Reload](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-spring-boot-restart-vs-reload) 章节所述, 使用两个类加载器实现了重启功能; 对于大多数应用程序, 这种方法很有效; 但是, 它有时会导致类加载问题  
默认情况下, IDE 中的任何打开项目都使用 "restart" 类加载器加载, 并且任何常规 `.jar` 文件都使用 "base" 类加载器加载; 如果是多模块项目, 并且不是每个模块都导入到 IDE 中, 则可能需要自定义内容; 为此, 你可以创建 `META-INF/spring-devtools.properties` 文件  
`spring-devtools.properties` 文件可以包含以 `restart.exclude` 和 `restart.include` 为前缀的属性; `include` 元素是被提取到 "restart" 类加载器中的项, 而 `exclude` 元素是被下推到 "base" 类加载器中的项; 属性的值是应用于类路径的正则表达式, 如以下所示
```
restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```
>所有属性键必须是唯一的, 只要属性以 `restart.include` 或者 `restart.exclude` 开头
>类路径中的所有 `META-INF/spring-devtools.properties` 都会被加载; 你可以将文件打包到项目中, 或者打包在项目使用的库中

##### 已知限制
对于使用标准的 `ObjectInputStream` 反序列化的对象, 重新启动功能会不起作用; 如果需要反序列化数据, 可能需要将 Spring 的 `ConfigurableObjectInputStream` 与 `Thread.currentThread().getContextClassLoader()` 结合使用
不幸的是, 一些第三方库反序列化没有考虑上下文类加载器; 如果你发现此类问题，则需要向原作者请求修复

##### 重新加载
`spring-boot-devtools` 模块包含一个内嵌的 LiveReload 服务器, 可用于在更改资源时触发浏览器刷新; LiveReload 浏览器扩展程序可从 [livereload.com](http://livereload.com/extensions/) 免费获取, 支持 Chrome, Firefox, Safari  
如果你不想在应用程序运行时启动 LiveReload 服务器, 则可以将 `spring.devtools.livereload.enabled` 属性设置为 `false`
>你一次只能运行一个 LiveReload 服务器; 在启动应用程序之前, 请确保没有其他 LiveReload 服务器在运行; 如果从 IDE 启动多个应用程序, 则只有第一个具有 LiveReload 支持

#### 全局设置
你可以通过将名为 `.spring-boot-devtools.properties` 的文件添加到 `$HOME` 文件夹来配置全局 devtools 设置 (请注意, 文件名以 "." 开头); 添加到此文件的任何属性都适用于计算机上使用 devtools 的所有 Spring Boot 应用程序; 例如, 要将 `restart` 配置总是使用 [触发器文件](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-devtools-restart-triggerfile), 你可以添加以下属性到 `~/.spring-boot-devtools.properties` 文件中
```
spring.devtools.reload.trigger-file=.reloadtrigger
```
>在 `.spring-boot-devtools.properties` 中激活的配置文件不会影响特定于配置文件的加载

#### 远程应用程序
Spring Boot 开发者工具不仅限于本地开发; 远程运行应用程序时, 你也可以使用一些功能; 远程支持是可选的, 要启用它, 你需要确保 devtools 包含在 repackage 的包中, 如下所示
```
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<configuration>
				<excludeDevtools>false</excludeDevtools>
			</configuration>
		</plugin>
	</plugins>
</build>
```
然后你需要设置 `spring.devtools.remote.secret` 属性, 如以下所示
```
spring.devtools.remote.secret=mysecret
```
>在远程应用程序上启用 `spring-boot-devtools` 存在安全风险; 你永远不应该在生产环境启用该支持

远程 devtools 支持分为两部分: 接收连接的服务器端端和在 IDE 中运行的客户端应用程序; 设置 `spring.devtools.remote.secret` 属性时, 将自动启用服务器组件; 客户端组件则必须手动启动

##### 运行远程客户端应用程序
远程客户端应用程序被设计为在从 IDE 中运行; 你需要运行 `org.springframework.boot.devtools.RemoteSpringApplication`, 其类路径与你连接的远程项目相同; 应用程序必需的一个参数是它所连接的远程 URL  
例如, 如果你使用的是 Eclipse 或 STS, 并且已部署名为 `my-app` 项目到Cloud Foundry, 那么你应执行以下操作
- 从 `Run` 菜单中选择 `Run Configurations…​`
- 创建一个新的 `Java Application` 发布配置
- 浏览 `my-app` 项目
- 使用 `org.springframework.boot.devtools.RemoteSpringApplication` 作为主类
- 添加 `https://myapp.cfapps.io` 到程序参数 (无论你的远程 URL 是什么)

正在运行的远程客户端可能类似于以下输出
```
.   ____          _                                              __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
\\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
'  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
=========|_|==============|___/===================================/_/_/_/
:: Spring Boot Remote :: 2.1.3.RELEASE

2015-06-10 18:25:06.632  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication on pwmbp with PID 14938 (/Users/pwebb/projects/spring-boot/code/spring-boot-devtools/target/classes started by pwebb in /Users/pwebb/projects/spring-boot/code/spring-boot-samples/spring-boot-sample-devtools)
2015-06-10 18:25:06.671  INFO 14938 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@2a17b7b6: startup date [Wed Jun 10 18:25:06 PDT 2015]; root of context hierarchy
2015-06-10 18:25:07.043  WARN 14938 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
2015-06-10 18:25:07.074  INFO 14938 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2015-06-10 18:25:07.130  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 0.74 seconds (JVM running for 1.105)
```
>因为远程客户端使用与真实应用程序相同的类路径, 所以它可以直接读取应用程序属性; 这也是如何读取 `spring.devtools.remote.secret` 属性并将其传递给服务器进行身份验证的方法
>始终建议使用 `https://` 作为连接协议, 以便加密流量避免被截获密码
>如果需要使用代理来访问远程应用程序, 请配置 `spring.devtools.remote.proxy.host` 和 `spring.devtools.remote.proxy.port` 属性

##### 远程更新
远程客户端以本地重新启动相同的方式监视应用程序类路径的更改; 任何更新的资源都会被推送到远程应用程序, 并且 (如果需要) 会触发重新启动; 如果你迭代使用本地没有的云服务的功能, 这将非常有用; 通常, 远程更新和重新启动比全部构建和再部署快得多
>仅在远程客户端运行时监视文件; 如果在启动远程客户端之前更改文件, 则不会将其推送到远程服务器

>**参考:**
[Developer Tools](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-devtools)
