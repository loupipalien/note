### 构建你的代码
Spring Boot 不需要任何特定的代码布局即可工作; 但这有一些最佳实践可以提供帮助

#### 使用 "默认包"
当类不包含包声明时, 它被认为是在 "默认包" 中; 通常不鼓励使用 "默认包", 这应该避免使用; 对于使用 `@ComponentScan`, `@EntityScan` 或 `@SpringBootApplication` 注解的 Spring Boot 应用程序, 这可能会导致特定问题, 因为每个 jar 中的每个类都将被读取
>我们建议你遵循 Java 推荐的包命名约定, 使用反向域名 (例, com.example.project)

#### 定位主应用程序类
我们通常建议你将主应用程序类放在其他类之上的根包中; ` @SpringBootApplication` 注解通常放在你的主类上, 它隐式地为某些项定义了一个基础 "搜索包"; 例如, 如果你正在编写 JPA 应用程序, 则使用带 `@SpringBootApplication` 注释类的所在包来搜索 `@Entity` 项; 使用根包还允许组件扫描仅应用于你的项目
>如果你不想使用 `@SpringBootApplication`, 那么导入 `@EnableAutoConfiguration` 和 `@ComponentScan` 注解会定义该行为, 因此你也可以此代替

以下清单展示了一个典型的布局
```
com
 +- example
     +- myapplication
         +- Application.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```
`Application.java` 文件将声明 `main` 方法以及基础的 `@SpringBootApplication`, 如下所示
```
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

>**参考:**
[Structuring Your Code](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-structuring-your-code)
