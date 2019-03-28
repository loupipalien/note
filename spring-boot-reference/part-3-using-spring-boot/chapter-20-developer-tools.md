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
