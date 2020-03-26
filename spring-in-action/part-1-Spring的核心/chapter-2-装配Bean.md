### 装配 Bean
创建应用对象之间协作关系的行为通常称为装配 (wiring), 这也是依赖注入 (DI) 的本质

#### Spring 配置的可选方案
- 在 XML 中进行显式配置
- 在 Java 中进行显式配置
- 隐式的 Bean 发现机制和自动装配

#### 自动化装配 Bean
Spring 从两个角度来实现自动化装配
- 组件扫描 (component scanning): Spring 会自动发现应用上下文中所创建的 Bean
- 自动装配 (autowiring): Spring 自动满足 Bean 之间的依赖

组件扫描和自动装配组合在一起就能发挥出强大的威力, 它们能够将显示配置降低到最少

##### 创建可被发现的 Bean
TODO
##### 为组件扫描的 Bean 命名
TODO
##### 设置组件扫描的基础包
TODO
##### 通过为 Bean 添加注解实现自动装配
TODO

#### 通过 Java 代码装配 Bean
相比于其他显示配置, JavaConfig 是更好的方案, 因为它更为强大, 类型安全并且对重构友好
##### 创建配置类
TODO
##### 声明简单的 Bean
TODO
##### 借助 JavaConfig 实现注入
TODO

#### 通过 XML 装配 Bean

##### 创建 XML 配置规范
TODO
##### 声明一个简单的 Bean
TODO
##### 借助构造器注入初始化 Bean
TODO
##### 设置属性
TODO

#### 导入和混合配置
在典型的 Spring 应用中, 可能会同时使用自动化和显示配置, 即便你更喜欢通过 JavaConfig 实现显示配置, 但有时候 XML 确实最佳方案; 好在 Spring 中, 这些配置方案都不是互斥的

##### 在 JavaConfig 中引用 XML 配置
- 使用 `@Import` 注解可以引入 JavaConfig
- 使用 `@ImportResource` 注解可以引入 XML 配置

##### 在 XML 配置中引用 JavaConfig
- 使用 `<import>` 标签可以引入 XML 配置
- 使用 `<bean>` 标签可以将 JavaConfig 导入
