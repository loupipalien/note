### 高级装配

#### 环境与 Profile

##### 配置 Pronfile Bean
- JavaConfig 配置 Profile
- XML 配置 Profile

##### 激活 Profile
Spring 在确定那个 Profile 处于激活状态时, 需要依赖两个独立的属性: `spring.profiles.active` 和 `spring.profiles.default`; 如果设置了 `spring.profiles.active` 属性的话, 那么它的值就会用来确定那个 Profile 是激活的; 如果没有设置 `spring.profiles.active` 的话, 那么 Spring 将会查找 `spring.profiles.default` 的值; 如果这两个参数都没有设置, 那就没有激活的 Profile, 因此只会创建那些没有定义在 Profile 中的 Bean; 有多种方式来设置这两个属性值
- 作为 `DispatcherServlet` 的初始化参数
- 作为 Web 应用的上下文参数
- 作为 JNDI 条目
- 作为环境变量
- 作为 JVM 的系统属性
- 在集成测试上, 使用 `@ActiveProfiles` 注解设置

#### 条件化的 Bean
Spring 4 引入了一个新的 `@Conditional` 注解, 它可以用到带有 `@Bean` 注解的方法上, 如果给定条件计算结果为 `true`, 就会创建这个 Bean, 否则的话这个 Bean 就会被忽略
```Java
@Bean
@Conditional(MagicExistsCondition.class)
public MagicBean MagicBean() {
    return new MagicBean();
}
```
设置给 `@Conditional` 的类可以是任意实现了 `Condition` 接口的类型
```Java
public interface Condition {
    boolean matches(ConditionContext ctxt, AnnotatedTypeMetadata metadata);
}
```
这个接口实现起来很简单, 只需提供 `matches()` 方法的实现即可, 返回 `true` 则创建 Bean 否则不创建; 另外参数 `ctxt` 和 `metadata` 中包含了很多信息, 基于这两个参数中的信息可以做出非常多的判定组合  
另外, Spring 4 中对 `@Profile` 注解进行了重构, 使其基于 `@Conditional` 注解和 `Condition` 接口实现
```Java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {
	/**
	 * The set of profiles for which the annotated component should be registered.
	 */
	String[] value();
}

//===============================================

class ProfileCondition implements Condition {
	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		if (context.getEnvironment() != null) {
			MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
			if (attrs != null) {
				for (Object value : attrs.get("value")) {
					if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
						return true;
					}
				}
				return false;
			}
		}
		return true;
	}
}
```

#### 处理自动装配的歧义性
在自动装配发生歧义的时候, Spring 提供了多种可选方案来解决这样的问题, 可以将可选 Bean 中的某一个设只为 `Primary` 的 Bean, 或者使用限定符 `Qualifier` 来帮助 Spring 将可选的 Bean 范围缩小到一个

##### 标示首选的 Bean
在 Spring 中, `@Primary` 注解用来表达最喜欢的方案, 这个注解可以与 `@Component` 等注解组合拥在组件扫描上, 也可以与 `@Bean` 组合用在 Java 配置的 Bean 声明中; XML 配置中的 `<bean/>` 标签有一个 `primary` 属性来指定首选的 Bean

##### 限定自动装配的 Bean
Spring 的限定符能够在所有可选的 Bean 上进行缩小范围操作, 最终能够达到只有一个 Bean 满足所规定的限制条件; 如果将所有的限定符都用上后依然存在歧义性, 那么可以继续使用更多的限定符来缩小范围  
`@Qualifier` 注解是使用限定符的主要方式, 可以与 `@Autowired` 和 `@Inject` 协同使用, 在注入的时候指定想要注入进入的是哪一个 Bean

###### 创建自定义的限定符
在 Spring 中可以为 Bean 设置自定义的限定符, 而不是依赖于将 Bean ID 作为限定符, 需要做的就是在 Bean 声明上添加 `@Qualifier` 注解; 在使用自定义 `@Qualifier` 值时, 最佳实践是为 Bean 选择特征性或描述性的述语, 而不是使用随意的名字

###### 使用自定义的限定符注解
面向特性的限定符要比基于 Bean ID 的限定符要好一些, 但是如果多个 Bean 都具有相同特性的话, 就又回到了不能限定到一个 Bean 范围的情况; 通常的思路是添加多个 `@Qualifier` 注解限定, 但是在 Java 8 以前不允许在同一个条目上重复出现相同类型的多个注解, Java 8 虽然允许但也要这个注解本身定义带有 `@Repeatable` 注解, 但 Spring 中 `@Qualifier` 定义没有添加 `@Repeatable` 注解; 所以为了避开这种限制, 可以基于 `@Qualifier` 注解自定义限定符注解
```Java
@Target({ElementType.CONSTRUCTOR, ElementType.FIELD, ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface ColdQualifier {}

//======================================

@Target({ElementType.CONSTRUCTOR, ElementType.FIELD, ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface CreamyQualifier {}
```
这样就有了两个不同类型但具有 `@Qualifier` 注解功能的限定符注解

#### Bean 的作用域
在默认情况下, Spring 应用上下文所有 Bean 都是作为单例 (singleton) 的形式创建的, 也就是说不管给定一个 Bean 被注入到其他 Bean 多少次, 每次注入的都是同一个实例  
Spring 定义了多种作用域, 可以基于这些作用域创建 Bean
- 单例 (Singleton)
在整个应用中, 只创建 Bean 的一个实例
- 原型 (Prototype)
每次注入或者通过 Spring 应用上下文获取的时候, 都会创建一个新的 Bean 的实例
- 会话 (Session)
在 Web 应用中, 为每个会话创建一个 Bean 实例
- 请求 (Request)
在 Web 应用中, 为每个请求创建一个 Bean 实例

可使用 `@Scope` 注解标记 Bean 的作用域, 在 XML 配置中的 `<bean/>` 标签中提供有 `scope` 的属性

##### 使用会话和请求作用域
TODO

##### 在 XML 中声明作用域代理
TODO

#### 运行时值注入
为了避免硬编码值, 而是想让这些值在运行时再确定; 为了实现这些功能, Spring 提供了两种在运行时求值的方式
- 属性占位符 (Property placeholder)
- Spring 表达式语言 (SpEL)

##### 注入外部的值
在 Spring 中, 处理外部值的最简单的方式就是声明属性源并通过 Spring 的 `Enviroment` 来检索属性
```Java
@Configuration
@PropertySource("classpath:app.properties")
public class ExpressiveConfig {
    @Autowired
    Enviroment env;

    @Bean
    public BlankDisc disc() {
        return new BlankDisc(env.getProperty("disc.title"), env.getProperty("disc.artist"));
    }
}
```

###### 深入学习 SPring 的 Enviroment
TODO

##### 解析属性占位符
Spring 一直支持将属性定义到外部的属性文件中, 并使用占位符将其值插入到 Spring Bean 中; 在 Spring 装配中, 占位符的形式为使用 `${...}` 包装的属性名称; 在 XML 配置中可以按照如下方式解析 `BlankDisc` 构造器参数
```
<bean id = "blankDisc" class = "com.demo.BlankDisc" c:_title = "${disc.title}" c:_artist = "${disc.artist}"/>
```
在 JavaConfig 中可以使用 `@Value` 注解
```Java
public BlankDisc(@Value("${disc.title}") String title, @Value("${disc.artist}") String artist) {
    this.title = title;
    this.artist = artist;
}
```
为了使用占位符, 必须要配置一个 `PropertyPlaceholderConfigurer` 的 Bean 或者 `PropertySourcePlaceholderConfigurer` 的 Bean; 从 Spring 3.1 开始, 推荐使用 `PropertySourcePlaceholderConfigurer`, 因为这个类能够基于 Spring `Enviroment` 及其属性源来解析占位符; 使用 JavaConfig 方式配置如下
```Java
@Bean
public static PropertySourcePlaceholderConfigurer placeHolderConfigurer() {
    return new PropertySourcePlaceholderConfigurer();
}
```
在 XML 配置中, Spring `context` 命名空间中的 `<context:property-placeholder/>` 元素会生成 `PropertySourcePlaceholderConfigurer` 的 Bean

##### 使用 Spring 表达式语言进行装配
Spring 3 引入了 Spring 表达式语言 (Spring Expression Language, SpEL), 它能够以一种简洁和强大的方式将值配置到 Bean 属性和构造器中, 在这个过程中所使用的额表达式会在运行时计算得到该值; SpEL 有很多特性, 包括
- 使用 Bean ID 来引用 Bean
- 调用方法和访问对象的属性
- 对值进行算术, 关系, 逻辑运算
- 正则表达式匹配
- 集合操作

SpEL 表达式要放在 `#{...}` 中

TODO
