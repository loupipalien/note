### Spring Beans 和依赖注入
你可以自由地使用任何标准的 Spring 框架技术来定义 bean 及其注入的依赖; 为简单起见, 我们经常发现使用 `@ComponentScan`(找到你的 bean) 并使用 `@Autowired` (做构造函数注入) 效果很好  
如果按照上面的建议构建代码 (在根包中定位应用程序类), 则可以添加不带任何参数的 `@ComponentScan`; 所有应用程序组件 (`@Component`, `@Service`, `@Repository`, `@Controller` 等) 都会自动注册为 Spring Beans  
以下示例展示了一个 `@Service` Bean, 它使用构造函数注入来获取所需的 `RiskAssessor` bean
```
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

	private final RiskAssessor riskAssessor;

	@Autowired
	public DatabaseAccountService(RiskAssessor riskAssessor) {
		this.riskAssessor = riskAssessor;
	}

	// ...

}
```
如果 bean 只有一个构造函数, 则可以省略 `@Autowired`, 如以下所示
```
@Service
public class DatabaseAccountService implements AccountService {

	private final RiskAssessor riskAssessor;

	public DatabaseAccountService(RiskAssessor riskAssessor) {
		this.riskAssessor = riskAssessor;
	}

	// ...

}
```
>请注意如何使用构造函数注入标记为final 的 riskAssessor 字段, 这也表示随后无法更改它

>**参考:**
[Spring Beans and Dependency Injection](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#using-boot-spring-beans-and-dependency-injection)
