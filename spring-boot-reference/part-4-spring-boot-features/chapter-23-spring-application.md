### SpringApplication
`SpringApplication` 类提供了一种方便的方法来引导从 `main()` 方法启动的 Spring 的应用程序; 在许多情况下, 你可以委托静态 `SpringApplication.run` 方法, 如下所示
```
public static void main(String[] args) {
	SpringApplication.run(MySpringConfiguration.class, args);
}
```
当你的应用程序启动时, 应该看到类似以下输出的内容
```
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::   v2.1.3.RELEASE

2013-07-31 00:08:16.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2013-07-31 00:08:16.166  INFO 56603 --- [           main] ationConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2014-03-04 13:09:54.912  INFO 41370 --- [           main] .t.TomcatServletWebServerFactory : Server initialized with port: 8080
2014-03-04 13:09:56.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)
```
默认的, 会显示 `INFO` 日志记录, 包括一些相关的启动详细信息, 例如启动应用程序的用户; 如果需要 `INFO` 以外的日志级别, 可以按照 [第 26.4 章节的 "日志级别"](v) 中的说明进行设置

#### 启动失败
如果您的应用程序无法启动, 则已注册的 `FailureAnalyzers` 有机会提供错误消息和具体操作来解决问题; 例如, 如果你在端口 `8080` 上启动 `Web` 应用程序并且该端口已在使用中, 应该会看到类似于以下消息的内容
```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```
>Spring Boot 提供了许多 FailureAnalyzer 实现, 你可以添加自己的

如果故障分析器没能够处理异常, 你仍然可以显示完整的条件报告, 以便更好地了解出现了什么问题; 为此, 你需要为 `org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener` [启用 `debug` 属性](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-external-config) 或 [启用 `DEBUG` 日志记录](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-custom-log-levels)  
如果使用 `java -jar` 运行应用程序, 则可以按如下方式启用 `debug` 属性
```
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

#### 自定义 Banner


>**参考:**
[SpringApplication])(https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#boot-features-spring-application)
