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
最理想的参数数量是零 (零参数函数), 其次是一 (单参数函数), 再次是二 (双参数函数), 应尽量避免三 (三参数函数); 有足够的特殊理由才能用三个以上的参数 (多参数函数)
