### 类

#### 类的组织
遵循标准的 Java 约定, 类应该从一组变量列表开始; 如果有公共静态常量, 应该先出现, 然后是私有静态变量, 以及私有实体变量, 很少会有公共变量; 公共函数应该跟在变量列表之后, 某个公共函数调用的私有工具函数紧随在该公共函数后面, 遵循自顶向下的原则

##### 封装
保持变量和工具函数的私有性, 但并不执着于此, 有时也需要用到受保护的 (protected) 变量和工具函数; 总是应该想办法使之保护隐私, 放松封装总是下策

#### 类应该短小
关于类的第一条规则是类应该短小, 第二条规则还要更短小; 对于函数通过计算代码行数衡量大小, 对于类衡量方法采用计算权责

##### 单一权责原则
单一权责原则 (SRP) 认为, 类或模块应有且只有一条加以修改的理由; 该原则即给出了权责的定义, 又是关于类的长度的指导方针; SRP 是 OO 设计中最为重要的概念之一, 也是较为容易理解和遵循的概念之一, 但往往也是最容易被破坏的类设计原则; 系统应该由许多短小的类而不是少量巨大的类组成, 每个小类封装一个权责, 只有一个修改的原因, 并与少数其他类一起协同达成期望的系统行为

##### 内聚
类应该只有少量实体变量; 类中的每个方法都应该操作一个或多个这种变量; 通常而言, 方法操作的变量越多, 就越黏聚到类上; 如果一个类中的每个变量都被每个方法所用, 则称该类具有最大的内聚性; 通常创建这种极大化的内聚类是不太可能的也不可取, 但是希望内聚性能保持在一个较高的水平, 内聚性越高, 意味着类中的方法和变量互相依赖, 互相结合成为一个整体; 保持函数和参数列表短小的策略, 有时会导致为一组子集方法所用的实体变量数量增加, 出现这种情况往往意味着至少有一个类要从大类中挣扎出来, 应该尝试将这些变量和方法拆分到两个或多个类中, 让新的类更加内聚


##### 保持内聚性就会得到许多短小的类
仅仅是将较大的函数拆分为较小的函数, 就将导致更多的类出现; 而保持参数列表的策略又会导致更多的实体变量; 而多个实体变量和短小的函数往往意味着类的内聚性下降, 这时应该将类拆分为多个更小的更内聚的类; 所以将大函数拆解为小函数的时机也是将类拆分为多个小类的时机; 这样程序会更加有组织, 也会拥有更为透明的结构

TODO...
```
PrintPrimes in Literate Programming
```

#### 为了修改而组织
对于多数系统修改将一直持续, 每处修改都让人冒着系统其他部分不能如期望般工作的风险; 在整洁的系统中, 应当对类加以组织, 以降低修改的风险   
以下代码中的 Sql 类用来生成提供恰当元数据的 SQL 格式化字符串, 这个类还没有写完, 所以暂时还不支持 update 语句等 SQL 功能; 当需要 Sql 类支持 update 语句时, 就得打开这个类进行修改, 打开类带来的问题是风险随之而来, 对类的任何修改都有可能破坏类中的其他代码, 必须全面重新测试
```
public class Sql {
    public Sql (String table, Column[] columns)
    public String create()
    public String insert(Object[] fields)
    public String selectAll()
    public findByKey(String keyColumn, String keyValue)
    public select(Column column, String pattern)
    public select(Criteria criteria)
    public preparedInsert()
    private String columnList(Column[] columns)
    private String valuesList(Object[] fields, final Column[] columns)
    private String selectWithCriteria(String criteria)
    private String placeholderList(Column[] columns)
}
```
当要增加一种新语句时就要修改 Sql 类, 改动单个语句类型时也要进行修改, 比如打算让 select 功能支持子查询; 存在两个修改理由, 说明 Sql 违反了 SRP 原则  
出现只与类的一小部分有关的私有方法行为, 意味着存在着改进空间; 但展开行动的基本动因则应该是系统的变动; 若认为 Sql 类在逻辑上已具足, 则无需担心对权责的拆分; 但如果在可预见的未来无需增加 update 功能, 就不该去动 Sql 类; 但一旦决定打开类就应当修正设计方案  
将以上 Sql 类的每个接口方法都重构到从 Sql 类派生出来的类中, 非公共的私有方法移动到需要的类中, 公共的私有方法则划分出两个独立的工具类
```
abstract public class Sql {
    public Sql(String table, Column[] columns);
    abstract public String generate();
}

public class CreateSql extends Sql {
    public CreateSql(String table, Column[] columns)
    @Override
    public String generate()
}

public class SelectSql extends Sql {
    public SelectSql(String table, Column[] columns)
    @Override
    public String generate()
}

public class InsertSql extends Sql {
    public InsertSql(String table, Column[] columns, Object[] fields)
    @Override
    public String generate()
}

public class SelectWithCriteriaSql extends Sql {
    public SelectWithCriteriaSql(String table, Column[] columns, Criteria criteria)
    @Override
    public String generate()
}

public class SelecteWithMatchSql extends Sql {
    public SelecteWithMatchSql(String table, Column[] columns, Column column, String pattern)
    @Override
    public String generate()
}

public class FindByKeySql extends Sql {
    public FindByKeySql(String table, Column[] columns, String keyColumn, String keyValue)
    @Override
    public String generate()
}

public class PreparedInsertSql extends Sql {
    public PreparedInsertSql(String table, Column[] columns)
    @Override
    public String generate()
    private String placeholderList(Column[] columns)
}

public class Where {
    public Where(String criteria)
    public String generate()
}

public class ColumnList {
    public ColumnList(Column[] columns)
    public String generate()
}
```
现在每个类都变得非常简单而且便于理解; 因为类与类相互之间隔离, 修改一个函数对其他函数影响的风险也降低; 更重要的是, 当需要增加 update 语句时, 现存类无需做任何修改, 只需要新增 继承 Sql 的 UpdateSql 类即可; 重新架构的 Sql 逻辑百利利而无一弊; 它支持 SRP, 也支持开放闭合原则 (OCP): 类应当对扩展开放, 对修改封闭; 通过子类化手段, 重新架构 Sql 类对添加新功能是开放的, 而且可以同时不触及其他类; 在理想的系统中, 希望通过扩展系统而非修改现有代码来添加新特性

##### 隔离修改
需求会改变, 所以代码也会改变; 具体类应包含实现细节, 而抽象类则只应呈现概念; 依赖具体细节的客户类, 当细节改变时就会有风险; 可以借助接口和抽象类来隔离这些细节带来的影响  
假定构建一个依赖于外部的 TokyoStockExchange API 的 Portfolio 类, 代表投资组合的价值, 则测试用例就会受到价值查询的连带影响; 与其设计直接依赖于 TokyoStockExchange 的 Portfolio 类, 不如创建 StockExchange 接口, 其中只声明一个方法
```
public interface StockExchange {
    Money currrentPrice(String symbol);
}
```
然后设计 TokyoStockExchange 类来实现这个接口, 并确保 Portfolio 的构造器接受作为参数的 StockExchange 引用
```
public class Portfolio {
    private StockExchange exchange;
    public Portfolio(StockExchange exchange) {
        this.exchange = exchange;
    }
    ...
}
```
这样就可以为 StockExchange 接口创建可测试的尝试性实现了, 该尝试性实现将返回固定的现值; 如果测试中购买 5 股微软股票, 则尝试性实现总是返回了每股 100 美元的现值; 对于 StockExchange 接口的尝试性实现简化为简单的表格查找; 然后再编写一个总投资价值为 500 美元的测试
```
public class PortfolioTest {
    private FixedStockExchangeStub exchange;
    private Portfolio portfolio;

    @Before
    protected void setUp() throws Exception {
        exchange = new FixedStockExchangeStub();
        exchange.fix("MSFT", 100);
        portfolio = new Portfolio(exchange);
    }

    @Test
    public void GivenFiveMSFTTotalShouldBe500() throws Exception {
        portfolio.add(5, "MSFT");
        Assert.assertEquals(500, portfolio.value());
    }
}
```
如果系统解耦到足以这样测试的程度, 也就更加灵活, 更加可复用; 部件之间解耦代表着系统中的元素互相隔离的很好; 隔离也让系统中每个元素的理解变得更加容易; 通过降低连接度, 类将遵循了另一条设计原则, 依赖倒置原则 (Dependency Inversion Principle, DIP); 本质而言, DIP 认为类应当依赖于抽象而不是依赖于具体细节
