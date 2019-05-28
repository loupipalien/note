### Bean 定义继承
bean 定义可以包含许多配置信息, 包括构造函数参数, 属性值和特定于容器的信息, 例如初始化方法, 静态工厂方法名称等; 子 bean 定义从父定义继承配置数据; 子定义可以根据需要覆盖某些值或添加其他值; 使用父 bean 和子 bean 定义可以节省大量的输入; 实际上, 这是一种模板形式  

如果以编程方式使用 `ApplicationContext` 接口, 则子 bean 定义由 `ChildBeanDefinition` 类表示; 大多数用户不在这个级别上使用它们, 而是以类似 `ClassPathXmlApplicationContext` 的方式声明性地配置 bean 定义; 使用基于 XML 的配置元数据时, 可以使用 `parent` 属性指定子 bean 定义, 并将父 bean 指定为此属性的值
```
<bean id="inheritedTestBean" abstract="true" class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```
如果没有指定, 则子 bean 定义使用父 bean 定义的配置, 但也可以覆盖它; 在后一种情况下, 子 bean 类必须与父类兼容, 也就是说, 它必须接受父类的属性值  
子 bean 定义从父级继承范围, 构造函数参数值, 属性值和方法覆盖, 并带有添加新值的选项; 你指定的任何范围, 初始化方法, 销毁方法或静态工厂方法设置都将覆盖相应的父设置  
其余设置始终取自子定义: depends-on, autowire mode, dependency check, singleton, lazy init  
前面的示例通过使用 `abstract` 属性将父 bean 定义显式标记为抽象的, 如果父定义未指定类, 则需要将父 bean 定义显式标记为 `abstract`, 如下所示
```
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```
父 bean 不能单独实例化, 因为它不完整, 并且也明确标记为抽象; 当定义是这样的抽象时, 它只能用作纯模板 bean 定义, 即作为子定义的父定义; 尝试使用这样一个抽象的父 bean, 通过将它作为另一个 bean 的 `ref` 属性引用或者使用父 bean `id` 进行显式的 `getBean()` 调用, 会返回错误; 类似地, 容器的内部 `preInstantiateSingletons()` 方法忽略定义为 `abstract` 的 bean 定义  
>`ApplicationContext` 默认情况下预先实例化所有单例; 因此, 重要的是 (至少对于单例 bean), 如果你有一个 (父) bean定义, 你只打算用作模板, 并且这个定义指定了一个类, 你必须确保将 `abstract` 属性设置为 `true`, 否则应用程序上下文将实际 (尝试) 预先实例化抽象 bean

>**参考:**
[Bean definition inheritance](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/beans.html#beans-child-bean-definitions)
