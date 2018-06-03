### 系统

#### 如何建造一个城市
掌管一切细节大概不行; 即便是管理一个即存的城市, 也是一个人无法做到的; 不过城市依然运转良好, 这是因为每个城市都有一组人管理不同的部分, 供水, 交通, 执法, 立法等等; 有的人负责全局, 其他人负责细节; 城市能运转, 还因为它演化出恰当的抽象等级和模块, 好让个人和他们管理的组件即便在不了解全局时也能有效的运转

#### 将系统的构造与使用分开
构造和使用是非常不一样的过程; 每个应用程序都该留意启始过程, 将构造和使用分离开来, 这样有利于打造良好的格式并强固系统; 对象构造的起始和设置过程也不例外, 应当将这个过程从正常的运行时逻辑分离开来, 确保拥有解决主要依赖问题的全局性一贯策略

##### 分解 main
将构造与使用分开的方法之一就是将全部构造过程搬迁到 main 或被称之为 main 的模块中, 设计系统的其余部分时可以假设对象都已正确构造和设置; main 函数创建系统所需的对象, 再传递给应用程序, 应用程序只管使用; 注意横贯 main 与应用程序之间隔离的依赖箭头的方向, 都是从 main 函数向外, 这表示应用程序对 main 或构造过程一无所知, 只简单的指望一切已齐备
```
    |------|      2: run(co)           |-------------|
    | main | ------------------------> | application |
    |------|                           |-------------|            
       |                                      |
    1: build                                  |
       |                                      |    
       V                                      V
  |---------|   1.1: construct    |-----------------------|
  | Builder | ------------------> | co: Configured Object |   
  |---------|     <<creates>>     |-----------------------|
```

##### 工厂
有时应用程序也要负责确定何时创建对象, 例如在某个订单处理系统中, 程序必须创建 LineItem 实体, 添加到 Order 对象; 在这种情况下, 可以使用抽象工厂模式让应用自行控制何时创建 LineItems, 但构造的细节却隔离于应用代码之外
```
        |------|     run(factory)        |-----------------|
        | main | ----------------------> | OrderProcessing | --------------|
        |------|                         |-----------------|               |
           |                                      |                        |
         create                                   |                        |
           |                                      |                        |
           V                                      V                        |
  |-----------------|                   |-----------------|                V
  | LineItemFactory |                   | <<interface>>   |           |----------|
  | implements      | ----------------> | LineItemFactory |           | LineItem |
  |-----------------|     <<creates>>   |-----------------|           |----------|
          |                             |  makeLineItem   |                ^
          |                             |-----------------|                |
          |----------------------------------------------------------------|                              
```
所以依赖都是从 main 指向 OrderProcessing 应用程序; 这代表应用程序与如何构建 LineItem 的细节是分离开来的; 构建能力由 LineItemFactoryImplementation 持有, 而 LineItemFactoryImplementation 又是在 main 这一边的; 但应用程序能完全控制 LineItem 实体何时构建, 甚至能传递应用特定的构造器参数

##### 依赖注入
有一种强大的机制可以实现分离构造与使用, 那就是依赖注入 (Dependency Injection, DI); 控制反转 (Inversion of Control, IoC) 是在依赖管理中额一种应用手段, 控制反转将第二权责从对象中拿出来, 转移到另外一个专注于此的对象中, 从而遵循了单一权责原则; 在依赖管理的情形中, 对象不应负责对自身依赖的实体化, 反之它应该将这份权重移交给其他 "有权力" 的机制, 从而实现控制的反转; 因为初始设置是一种全局问题, 这种授权机制通常要么是 main 例程, 要么是有特定目的的容器  
JNDI 查找是 DI 的一种 "部分" 实现, 在 JNDI 中对象请求目录服务提供一种符合特定名称的 "服务", `MyService myService = (MyService) (jndiContext.lookup("NameOfMyService"));`, 调用对象并不控制真正返回对象的类别 (当然前提是它实现了恰当的接口), 但调用对象仍然主动分解了依赖  
真正的依赖注入还要更进一步; 类并不直接分解其依赖, 而是完全被动的; 它提供可用于注入依赖的赋值器方法或构造器参数 (或二者皆有); 在构造过程中, DI 容器实体化需要的对象 (通常按需创建), 并使用构造器参数或赋值器方法将依赖连接到一起; 至于哪个依赖对象真正得到使用, 是通过配置文件或一个有特殊目的的构造模块中编程决定的; Spring 框架提供了最有名的 Java DI 容器, 用户在 XML 配置文件中定义互相关联的对象, 然后使用 Java 代码请求特定对象

#### 扩容
"一开始就做对系统" 纯属神话; 反之, 应该只去实现今天的用户故事, 然后重构, 明天再扩展系统实现新的用户故事; 这就是迭代和增量敏捷的精髓所在; 测试驱动开发, 重构以及它们打造出的整洁代码, 在代码层面保证了这个过程的实现; 与物理系统相比软件系统比较独特, 它们的架构都可以递增式地增长, 只要持续将关注面恰当的切分  
初始的 EJB1 和 EJB2 架构没有恰当的切分关注面, 从而给有机增长压上了不必要的负担; 以下是一个没有充分隔离关注问题的反例
```
// Bank EJB 的 EJB2 本地接口
public interface BankLocal extends java.ejb.EJBLocalObject {
    String getStreetAddr1() throws EJBException;
    String getStreetAddr2() throws EJBException;
    String getCity() throws EJBException;
    String getState() throws EJBException;
    String getZipCode() throws EJBException;
    void setStreetAddr1(String street1) throws EJBException;
    void setStreetAddr2(String street1) throws EJBException;
    void setCity(String city) throws EJBException;
    void setState(String state) throws EJBException;
    void setZipCode(String zip) throws EJBException;
    Collection getAccounts() throws EJBException;
    void setAccounts(Collection accounts) throws EJBException;
    void addAccount(AccountDTO accountDTO) throws EJBException;
}
```
以上列出了银行地址的几个属性, 和一组该银行拥有的账户, 其中每个账户的数据都由单独的 AccountEJB 所持有
```
// 相应的 EJB2 Entity Bean 实现
public abstract class Bank implements javax.ejb.EntityBean {
    public abstract String getStreetAddr1();
    public abstract String getStreetAddr2()
    public abstract String getCity();
    public abstract String getState();
    public abstract String getZipCode();
    public abstract void setStreetAddr1(String street1);
    public abstract void setStreetAddr2(String street1);
    public abstract void setCity(String city);
    public abstract void setState(String state);
    public abstract void setZipCode(String zip);
    public abstract Collection getAccounts();
    public abstract void setAccounts(Collection accounts);
    public abstract void addAccount(AccountDTO accountDTO) {
        InitialContext context = new InitialContext();
        AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
        AccountLocal account = accountHome.create(accountDTO);
        Collection accounts = getAccounts();
        accounts.add(account);
    }
    // EJB container logic
    public abstract void setId(Integer id);
    public abstract Integer getId();
    public Integer ejbCreate(Integer id) {...}
    public void ejbPostCreate(Integer id) {...}
    // The rest had to be implemented but were usually empty
    public void setEntityContext(EntityContext ctx) {}
    public void unsetEntityContext() {}
    public void ejbActivate() {}
    public void ejbPassivate() {}
    public void ejbLoad() {}
    public void ejbStore() {}
    public void ejbRemove() {}
  }
```
最后要编写一个或多个 XML 部署说明, 将对象相关映射细节指定给某个持久化存储空间, 说明期望的事物行为, 安全约束等; 业务逻辑与 EJB2 应用容器紧密耦合, 必须提供子类化容器类型, 必须提供多个该容器所需要的生命周期方法; 由于存在这种与重量级容器的紧耦合, 隔离单元测试就很困难, 有必要模拟出容器或者花费大量时间在真实服务器上部署 EJB 和测试; 也由于耦合的存在, 在 EJB2 架构之外的复用实际变得不可能

##### 横贯式关注面
持久化类关注面倾向于横贯某个领域的天然对象边界, 会想用同样的策略来持久化所有对象; 原则上可以从模块, 封装的角度推理持久化策略; 但在实践中, 却不得不将实现了持久化策略的代码铺展到许多对象中, 通常用术语 "横贯式关注面" 来形容这类情况; 同样的, 持久化框架和领域逻辑, 孤立的看可以是模块化的, 问题在于横贯这些领域的情形  
EJB 架构处理持久化, 安全和事务的方法要早于面向方面编程 (aspect-oriented programming, AOP), 而 AOP 是一种恢复横贯式关注面模块化的普适手段; 在 AOP 中, 被称为方面的模块构造指明了系统中哪些点的行为会以某种一致的方式被修改, 从而支持某种特定的场景, 这种说明是用某种简洁的声明或编程机制来实现的

#### Java 代理
Java 代理适用于简单的情况, 例如在单独的对象或类中包装方法调用; 然而 JDK 提供的动态代理仅能与接口协同工作, 对于代理类得使用字节码操作库, 如 CGLIB, ASM, Javaassist 等
```
/**
* JDK 代理范例
*/

// Bank.java
public interface Bank {
    Collection<Account> getAccounts();
    void setAccounts(Collection<Account> accounts);
}

// BankImpl.java
public class BankImpl implements Bank {
    private List<Account> accounts;

    public Collection<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(Collection<Account> accounts) {
        this.accounts = new ArrayList<Account>();
        for (Account account: accounts) {
            this.accounts.add(account);
        }
    }
}

// BankProxyHandler.java
public class BankProxyHandler implements InvocationHandler {
    private Bank bank;

    public BankProxyHandler(Bank bank) {
        this.bank = bank;
    }

    // Method defined in InvocationHandler
    public Object invoke(Object proxy, Method method, Object[] objects) throws Throwable {
        String methodName = method.getName();
        if (methodName.equals("getAccounts")) {
            bank.setAccounts(getAccountsFromDatabase());
            return bank.getAccounts();
        } else if (methodName.equals("setAccounts")) {
            bank.setAccounts((Collection<Account>) args[0]);
            setAccountsToDatabase(bank.getAccounts());
            return null;
        } else {
            ...
        }
    }

    // Lost of detail here
    protected Collection<Account> getAccountsFromDatabase() {...}
    protected void setAccountsToDatabase(Collection<Account> accounts) {...}
}

// Somewhere else ...
Bank bank = (Bank) Proxy.newProxyInstance(Bank.class.getClassLoader(), new Class[] {Bnank.class}, new BankProxyHandler(new BankImpl());
```
Proxy API 需要一个 InvocationHandler 对象, 用来实现对代理的全部 Bank 方法调用, BankProxyHandler 使用 Java 反射 API 将一般方法调用映射到 BankImpl 中对应方法, 以此类推

#### 纯 Java AOP 框架
编程工具能自动处理大多数代理模板代码; 在数个 Java 框架中, 代理都是内嵌的, 从而能够以纯 Java 代码实现面向方面编程; 在 Spring 中, 将业务逻辑代码为旧式 Java 代码, POJO 自扫门前雪, 并不依赖企业框架; 使用描述性配置文件或 API, 把需要的应用程序构架组合起来, 包括持久化, 事务, 安全, 缓存, 恢复等横贯性的问题; 在许多情况下, 框架以对用户透明的方式处理使用 Java 代理或字节代码库的机制; 这些声明驱动了依赖注入容器, DI 容器再实体化主要对象, 并按需将对象连接起来
```
// Spring 2.x 的配置文件
<beans>
    ...
    <bean id = "appDataSource" class = "org.apache.commons.dbcp.BasicDataSource" destory-method = "close" p:dricerClassName = "com.mysql.jdbc.Driver" p:url = "jdbc:mysql://localhost:3306/mydb" p:username = "me" />

    <bean id = "bankDataAccessObject" class = "com.example.banking.persistence.BankDataAccessObject" p:dataSource-ref = "appDataSource" />

    <bean id = "bank" class = "com.example.banking.model.Bank" p:dataAccessObject-ref = "bankDataAccessObject" />
    ...
</beans>
```
在应用程序中, 只添加少数几行代码, 用来向 DI 容器请求系统中的顶层对象, 如 XML 文件中所定义的那样
```
XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```
只有区区几行代码与 Spring 相关的代码, 应用程序几乎完全与 Spring 分离, 消除了 EJB2 之类系统中那样紧耦合问题; 尽管 XML 可能会冗长且难以阅读, 配置文件中定义的 "策略" 还是要比那种隐藏在幕后自动创建的复杂的代理和方面逻辑来得更简单; EJB3 也很大程度上遵循了 Spring 通过描述性手段支持横贯式关注面的模型
```
// EJB3 版本的 Bank
@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;
    @Embeddable // An object inlined in Bank's DB row
    public class Address {
        protected String streetAddr1;
        protected String streetAddr2;
        protected String city;
        protected String state;
        protected String zipCode;
    }

    @Embedded
    private Address address;

    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy = "bank")
    private Collection<Account> accounts = new ArrayList<Account>();

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void addAccount(Account account) {
        account.setBank(this);
        accounts.add(account);
    }

    public Collection<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(Collection<Account> accounts) {
        this.accounts = accounts;
    }
}
```
以上代码比 EJB2 时已整洁许多, 有些实体细节仍然在 annotation 中存在; 但并没有任何信息超出 annotation 之外, 代码依然整洁, 清晰, 也易于测试驱动和维护; 如果愿意 annotation 中有些或全部持久化信息可以转移到 XML 部署描述中, 只留下真正的 POJO

#### AspectJ 的方面
通过方面来实现关注面切分额功能最全的工具是 AspectJ 语言, 一种提供一流的将方面作为模块构造处理支持的 Java 扩展, z在大部分用到方面特性的情况下, Spring AOP 和 JBoss AOP 提供的纯 Java 实现手段已足够, 然而 AspectJ 却提供了一套用于切分关注面的丰富而强有力的工具, AspectJ 的弱势在于需要采用几种新的工具, 学习新语言构造和使用方式

#### 测试驱动系统架构
最佳的系统架构由模块化的关注面领域组成, 每个关注面均用纯 Java (或其他语言) 对象实现, 不同领域之间用最不具有侵害性的方面或类方面工具整合起来; 这种架构能测试驱动, 就像代码一样

#### 优化决策
模块化和关注面切分成就了分散化管理和决策, 在巨大的系统中, 不管是一座城市或一个软件项目, 无人能做所有的决策; 拥有模块化关注面的 POJO 系统提供的敏捷能力, 允许基于最新的知识做出优化的, 时机刚好的决策, 决策的复杂性也降低了

#### 明智使用添加可论证价值的标准
TODO...

#### 系统需要领域特定语言
在软件领域, 领域特定语言 (Domain-Specific Language, DSL) 是一种单独的小型脚本语言或以标准语言写就的 API, 优秀的 DSL 填平了领域概念的代码之间的壕沟, 可以降低不正确将领域翻译为实现的风险

#### 小结
系统应该是整洁的, 侵害性架构会泯灭领域逻辑, 冲击敏捷能力; 当领域逻辑受到困扰, 代码质量也会堪忧, 因为缺陷更易隐藏, 用户故事更难实现; 当敏捷能力受到损害时, 生产力也会降低, TDD 的好处遗失殆尽
