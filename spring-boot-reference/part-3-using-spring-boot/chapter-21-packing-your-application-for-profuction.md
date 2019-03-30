### 为生产环境打包你的应用程序
可执行 jar 可用于生产部署; 由于它们是自包含的, 因此它们也非常适合基于云的部署  
对于其他 "生产就绪" 功能, 例如运行状况, 审计和度量 REST 或 JMX 端点, 请考虑添加 `spring-boot-actuator`; 详见 [第 V 章节 "Spring Boot Actuator: 生产就绪功能"](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#production-ready)

>**参考:**
[Packaging Your Application for Production](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-packaging-for-production)
