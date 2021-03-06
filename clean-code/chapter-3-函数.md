### 函数

#### 短小
**函数的第一规则就是要短小, 第二条规则还要更短小**

##### 代码块和缩进
if 语句, else 语句, while 语句等, 其中的代码块应该只有一行; 该行大抵应该是一个函数调用语句; 这样不但能保持函数短小, 而且因为块内函数调用的函数拥有较具有说明性的名称, 从而增加了文档上的价值; 这也意味着函数不应该大到足以容纳嵌套结构, 所以函数的缩进层数不该多于一层或两层; 这样的函数容易阅读和理解

#### 只做一件事
**函数应该只做一件事, 做好这件事, 只做这一件事**
如果函数只是做了该函数名下同一抽象层上的步骤, 则函数还是只做了一件事; 编写函数毕竟是为了把大一些的概念 (换而言之, 函数的名称) 拆分为另一抽象层上的一系列步骤; 判定函数是否不止做了一件事, 还有一个方法, 就是看是否能再拆出一个函数, 该函数不仅只是单纯地诠释其实现

#### 每个函数一个抽象层级
要确保函数只做一件事, 函数中的语句都要在同一抽象层级上; 函数中混杂不同的抽象层级, 往往使人迷惑; 读者可能无法判断某个表达式是基础概念还是细节; 更恶劣的是, 一旦细节与基础概念混杂, 更多细节就会在函数中纠结起来

##### 自顶向下读代码: 向下规则
想要每个函数后面都跟着位于下一抽象层级的函数, 这样一来, 在查看函数列表时, 就能循抽象层级向下阅读了; 换一种说法: 程序就像是一系列 TO 起头的段落, 每一段都描述当前抽象层级, 并引用位于下一抽象层级的后续 TO 起头段落

#### switch 语句
写出短小的 switch 语句很难; 写出只做一件事的 switch 语句也很难, switch 天生就要做 N 件事; 不幸的是我们总无法避开 switch 语句; 不过还是能够确保每个 switch 都埋藏在较低的抽象层级, 而且永远不重复, 可以使用多态来实现这一点
```
public class Payroll {
    // 呈现了依赖于雇员类型的工资计算操作
    public Money calculatePay(Employee e) throws InvaildEmployeeType {
        switch(e.type) {
            case COMMISSIONED:
                return calculateCommissionedPay(e);
            case HOURLY:
                return calculateHourlyPay(e);
            case SAlARIED:
                return calcualateSalariedPay(e);
            default:
                throw new InvaildEmployeeType(e.type);
        }
    }
}
```
该函数有好几个问题; 首先它太长, 当出现新的雇员类型时, 还会变得更长; 其次, 它明显不止做了一件事; 第三, 它违反了单一权责原则 (Single Responsibility Principle, SRP), 因为有好几个修改它的理由; 第四, 它违反了开放闭合原则 (Open Closed Principle, OCP), 因为每当添加新类型时, 就必须修改之; 不过该函数最麻烦的可能是到处皆是类似结构的函数, 例如可能有: isPayday(Employee e, Date date), deliverPay(Employee e, Money pay) 等等, 它们的结构都会有同样的问题  
该问题的解决方案是将 switch 语句埋到抽象工厂底下, 不让任何人看到; 该工厂使用 switch 语句为 Employee 的派生物创建适当的实体, 而不同的函数, calculatePay, isPayDay, deliverPay 等则由 Employee 接口多态地接受派遣
```
// Employee 抽象类
public abstract class Employee {
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay();
}

// EmployeeFactory 接口类
public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvaildEmployeeType;
}

// EmployeeFactory 实现类
public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvaildEmployeeType {
        switch(r.type) {
            case COMMISSIONED:
                return CommissionedEmployee(r);
            case HOURLY:
               return HourlyEmployee(r);
            case SAlARIED:
                return SalariedEmployee(r);
           default:
               throw new InvaildEmployeeType(e.type);
        }
    }
}
```

#### 使用描述性的名称
沃德原则: 如果每个例程都让你感到深合已意, 那就是整洁代码; 要遵循这一原则的大半工作都在于为只做一件事的小函数取个好名字; 函数越短小, 功能越集中, 就越便于取个好名字  
别害怕长名称, 长而具有描述性的名称要比短而令人费解的名称好; 长而具有描述性的名称要比描述性的长注释好; 使用某种命名约定, 让函数名称中的多个单词容易阅读, 然后使用这些单词给函数取个能说清楚其功用的名称; 选择描述性的名称能理清关于模块的设计思路, 追索好名称往往导致对代码的改善重构; 命名方式要保持一致, 使用与模块名一脉相承的短语, 名词或动词给函数命名

#### 函数参数
最理想的参数数量是零 (零参数函数), 其次是一 (单参数函数), 再次是二 (双参数函数), 应尽量避免三 (三参数函数); 有足够的特殊理由才能用三个以上的参数 (多参数函数); 参数与函数名处在不同的抽象层级, 它要求你了解目前并不特别重要的细节

##### 一元函数的普遍形式
向函数传入单个参数有两种极普遍的理由: 也许会问关于那个参数的问题, 就像在 boolean fileExists("FileName") 中那样; 也可能是操作该参数, 将其转换为其他的什么东西再输出, 例如 InputStream fileOpen("FileName") 把 String 类型的文件名文件转换为 InputStream 类型的输出; 这就是读者看到函数时所期待的东西, 所以应当选用较能区别这两种理由的名称, 而且总在一致的上下文中使用这两种形式; 还有一种虽然不那么普遍但是极为有用的单参数函数形式, 那就是事件 (event); 在这种形式中有输入参数而无输出参数, 程序将函数看作是一个事件, 使用该参数修改系统的状态, 例如 void passwordAttemptFailedNtimes(int attempts); 小心使用这种形式, 应当让读者了解到这是个事件, 谨慎的选用名称和上下文语境

##### 标识参数
标识参数丑陋不堪, 向函数传入布尔值简直就是骇人听闻的做法; 这样做方法签名立刻变得复杂起来, 大声宣布本函数不止做一件事; 如果标识为 true 将会这样做, 标识为 false 则会那样做

##### 二元函数
有两个参数的函数要比一元函数难懂, 例如 wirteFiled(name) 和 writeField(outputStream, name) 好懂; 二元函数不算恶劣, 使用二元函数时尽量小心; 应该尽量利用一些机制将其转换为一元函数; 例如可以把 writeField 方法写成 outputStream 的成员之一, 从而可以 outputStream.writeField(name) 这样使用; 或者也可以把 outputStream 写成当前类的成员变量, 从而无须传递它; 还可以分离出类似 FieldWriter 的新类, 在其构造器中采用 outputStream, 并且包含一个 write 方法

##### 三元函数
有三个参数的函数要比二元函数难懂的多, 排序, 琢磨, 忽略的问题都会在此加倍体现

##### 参数对象
如果函数看来需要两个, 三个或更多的参数, 就说明其中一些参数应该封装成类了: Circle makeCircle(double x, double y, double radius) 重构为 Circle makeCircle(Point center, double radius)

##### 参数列表
有时想要向函数传入数量可变的参数, 如 String.format("&s worked %0.2f hours.", name, hour)

##### 动词与关键字
给函数取个好名字, 能够较好的解释函数的意图, 以及参数的顺序和意图; 对于一元函数, 函数名与参数应当形成一种非常良好的动词/名词对形式; 例如: write(name) 或 wirteField(name)

#### 无副作用
副作用是一种谎言. 函数承诺只做一件事, 但是还是会做其他的被藏起来的事; 但有时它会对自己类中的变量做出未能预期的改动, 例如会把变量搞成函数传递的参数或者是系统全局变量, 无论哪种情况都是具有破坏性的, 会导致古怪的时序性耦合及顺序依赖
```
public class UserValidator {
    private Cryptographer cryptographer;

    public boolean checkPassword(String userName, String password) {
        User user = UserGateway.findByName(userName);
        if (user != User.NULL) {
            String codedPhrase = user.getPhraseEncodedByPassword();
            String phrase = cryptographer.decrypt(codedPhrase, password);
            if ("Valid Password").equals(phrase) {
                Session.initialize();
                return true;
            }
        }
        return false;
    }
}
```
以上函数的副作用就在于 Session.initialize() 的调用; checkPassword 函数就是用来检查密码的, 该函数并未暗示它会初始化该会话, 所以当某个误信了函数名的调用者想要检查用户有效性时, 就得冒着抹除现有会话数据的风险; 这一副作用造出了一次时序性耦合, 也就是说 checkPassword 至能在特定时刻调用 (即在初始化会话是安全的时候调用); 如果在不合适的时候调用, 会话数据就有可能沉默的丢失; 时序性使人迷惑, 特别是当它躲在副作用后面的时候

##### 输出参数
面向对象语言中对输出参数的大部分需求已经消失了, 因为 this 也有输出函数的以为意味在内; 普遍而言应该避免使用输出参数, 如果参数必须要修改某种状态, 就修改所属对象的状态吧

#### 分隔指令和询问
函数要么做什么事, 要么回答什么事, 二者不可兼得; 函数应该修改某对象的状态, 或者是返回该对象的有关信息; 两者都干常会导致混乱, 如以下的例子
```
public boolean set(Stringa attribute, String value);
```
该函数设置某个指定属性, 如果成功就返回 true, 如果不存在那个属性则返回 false; 这样就导致了以下这样的语句
```
if (set("username", "unclebob")) {
    ...
}
```
从读者的角度考虑下, 这是什么意思呢? 是在问 username 属性值是否之前已经设置为 unclebob 吗? 还是在问 username 属性值是否成功设置为 unclebob 呢? 从这一行调用很难判断其含义, 因为 set 是动词还是形容词并不清楚

#### 使用异常替代返回错误码
从指令式函数返回错误码轻微违反了指令与询问分隔的规则, 它鼓励了在 if 语句判断中把指令当作表达式的使用, `if (deletePage(page) == E_OK)`, 这不会引起动词/形容词混淆, 但却导致更深层次的嵌套结构, 当返回错误码时, 就是在要求调用者立刻处理错误; 如果使用异常替代返回错误码, 错误处理代码就能从主路径代码中分离出来, 得到简化

##### 抽离 Try/Catch 代码块
Try/Catch 代码块丑陋不堪, 它们搞乱了代码结构, 把错误处理和正常流程混为一谈, 最好把 try 和 catch 代码块的主体部分抽离出来, 另外形成函数
```
public void delete(Page page) {
    try {
        // 抽离为一个函数
        deletePageAndAllReferences(page);
    } catch(Exception e) {
        logger.error(e.getMessage());
    }
}
```

##### 错误处理就是一件事
函数应该只做一件事, 错误处理就是一件事; 因此处理错误的函数不该做其他事, 这意味着如果关键字 try 在某个函数中存在, 它就该是这个函数的第一个单词, 而且在 catch/finally 代码块后也不该有其他内容

##### Error.java 依赖磁铁
返回错误码通常暗示某处有个类或者是枚举, 定义了所有错误代码
```
public enum Error {
    OK, INVALID, NO_SUCH, LOCKED, OUT_OF_RESOURCES, WAITING_FOR_EVENT;
}
```
这样的类就是一块依赖磁铁 (dependency magnet); 其他许多类都得导入和使用它, 当 Error 枚举类修改时, 所有这些其他的类都需要重新编译和部署, 这对 Error 类造成了负面压力, 程序员不愿意增加新的代码, 因为这样就得重新构建和部署所有东西, 于是就复用旧的错误码, 而不添加新的; 使用异常替代错误码, 新异常就可以从异常类派生出来, 无须重新编译或重新部署

#### 别重复自己
重复可能是软件中一切邪恶的根源; 许多原则和实践规则都是为了控制与消除重复而创建; 例如全部的考德 (Codd) 数据库范式都是为了消灭数据重复而服务, 面向对象编程中将代码集中到基类从而避免冗余; 面向切面编程 (Aspect Oriented Programming) 和面向组件编程 (Component Oriented Programming) 多少也是消除重复的一种策略

#### 结构化编程
Edsger Dijkstra 的结构化编程规则认为: 每个函数, 函数中的代码块都应该只有一个入口, 一个出口; 遵循这些规则, 就意味着在每个函数中只该有一个 return 语句, 循环中不能有 break 或 continue 语句, 而且永远不能有任何 goto 语句; 但这些规则在大函数中才有明显的好处, 在小函数中益处不多; 所以只要函数保持的短小, 偶尔出现 return, break, continue 语句没有什么坏处, 甚至更具有表达力

#### 如何写出这样的函数
打磨代码, 分解函数, 修改名称, 消除重复, 测试覆盖

#### 小结
把系统当作故事来讲, 而不是当作程序来写; 使用选定编程语言提供的工具构建一种更为丰富且更具有表达力的语言, 用来讲故事; 那种领域特定语言的一部分, 就是描述在系统中发生的各种行为的函数层级; 在一种狡猾的递归操作中, 这种行为使用它们定义的与领域紧密相关的语言讲述自己的那个小故事
