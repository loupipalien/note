### 类和接口

#### 第 13 条: 使类和成员的可访问性最小化
要区别设计良好的模块与设计不好的模块, 最重要的因素在于这个模块对于外部的其他模块而言, 是否隐藏器内部数据和其他实现细节; 设计良好的模块会隐藏所有的实现细节, 把它的 API 与它的实现清晰地隔离开来, 然后模块之间只通过它们的 API 进行通信, 一个模块不需要知道其他模块的内部工作情况; 这个概念被称为**信息隐藏 (information hiding) 或封装**, 是软件设计的基本原则之一  

##### 尽可能的使每个类或者成员不被外界访问
对于顶层的 (非嵌套的) 类和接口, 只有两种可能的访问级别: 包私有的 (package-private) 和公有的 (public); 如果类或者接口能够被做成包私有的, 那么它就应该被做成包私有的; 通过把类或者接口做成包私有的, 它实际上成了这个包的实现的一部分, 而不是包导出 API 的一部分, 在以后的发行版本中, 可以对它进行修改, 替换, 删除, 而无需担心会影响到现有的客户端程序, 如果把它做成公有的, 就有责任永远支持它, 并保持它们的兼容性  
如果一个包级私有的顶层类 (或者接口) 只是在一个类的内部被用到, 就应该考虑使它成为唯一使用它的那个类的私有嵌套类, 这样可以将它的可访问范围从包中的所有类缩小到了使用它的那个类; 然而降低不必要公有类的可访问性比降低包级私有的顶层类更重要的多, 因为公有类是包的 API 的一部分, 而包级私有的顶层类是这个包实现的一部分; 对于成员 (域, 方法, 嵌套类, 嵌套接口) 有四种访问级别, 按照以下顺序依次递增
- 私有的 (private): 只有在声明该成员的顶层类内部才可以访问这个成员
- 包级私有的 (package-private): 声明该成员的包内存不的任何类都可以访问这个成员
- 受保护的 (protected): 声明该成员的类的子类可以访问这个成员, 并且声明该成员的包内部的任何类也可以访问这个成员
- 公有的 (public): 在任何地方都可以访问该成员

私有成员和包级私有成员都是一个类的实现中的一部分, 一般不会影响它的导出 API, 然而如果这个类实现了 Serializable 接口, 这些域就有可能会被 "泄露" 到导出 API 中; 对于公有类的成员, 当访问级别从包级私有变成保护级别时会大大增强可访问性, 但也意味着其变成类的导出 API 的一部分, 必须永远得到支持; 有一条规则限制了降低方法的可访问性的能力, 即方法覆盖了超类中的一个方法, 子类中的访问级别就不允许低于超类中的访问级别  
实例域绝不能是公有的, 如果域是非 final 的或者是一个指向可变对象的 final 引用, 那么一旦使这个域成为公有的, 就放弃了对存储在这个域中的值进行限制的能力, 因此包含可变域的类不是线程安全的; 同样的建议也适用于静态域, 只有公有的静态 final 域暴露的常量 (基本类型的值或者不可变对象) 是例外情况

#### 在公有类中使用访问方法而非公有域
如果类可以在它所在的包的外部进行访问就提供访问方法; 如果类是包级私有的或者私有的嵌套类, 直接暴露它的数据域并没有本质的错误

#### 使可变性最小化
不可变类只是其实例不能被修改的类, 每个实例中包含的所有信息都必须在创建该实例的时候就提供, 并在对象的整个生命周期内固定不变, Java 类库中有基本类型包装类, String, BigInteger, BigDecimal 等; 不可变类比可变类更易设计, 实现, 使用, 安全; 使类成为不可变类要遵守以下五条规则
- 不要提供任何会修改对象状态的方法 (也称为 mutator)
- 保证类不会被扩展: 这样可以防止粗心或者恶意的子类假装对象的状态已经转变而破坏类的不可变行为, 为了防止子类化一般的做法是将这个类成为 final 的
- 使所有的域都是 final 的: 通过系统的强制方式保证线程安全的
- 使所有的域都成为私有的: 这样可以防止客户端或者访问被域引用的可变对象的权限. 并防止客户端直接修改这些对象
- 确保对于任何可变组件的护持访问: 如果类具有指向可变对象的域, 则必须确保该类的客户端无法获得指向这些对象的引用, 并且永远不要用客户端提供的对象引用来初始化这样的域, 也不要从任何访问方法中返回该对象的引用; 在构造器, 访问方法, readObject 方法中使用保护性拷贝 (defensive copy) 技术

不可变对象的本质上是线程安全的, 它们不要求同步, 所以不可变对象可以被自由地共享, 由此导致的结果是不可变类永远也不需要进行保护性拷贝, 因为这些拷贝始终等于原始对象, 因此不需要也不应该为不可变类提供 clone 方法或者拷贝构造器 (虽然 String 类具有拷贝构造器, 但是应该尽量少用); 不可变对象甚至也可以共享它们的内部信息; 不可变对象也为其他对象提供了大量的构件; 不可变类的唯一缺点是对于不同的值需要一个单独的对象  
对于不可变类的序列化需要注意, 不可变类实现了 Serializable 接口, 并且它包含一个或者多个指向可变对象的域, 就必须提供一个显式的 readObject 或者 readResolve 方法, 或者使用 ObjectOutputStream.writeUnrshared 和 ObjectInputStream.readUnshared 的方法, 即使默认的序列化的形式是可以接受的, 否则攻击者可能从不可变类创建可变的实例

#### 复合优先于继承
继承是实现代码重用的有力手段, 但它并非永远是完成这项工作的最佳工具; 在包内使用继承是非常安全的, 对于专门设计用于继承的且有良好说明文档的类来说也是安全的, 但对普通的具体类进行跨越包边界的继承是非常危险的  
与方法调用不同的是, 继承打破了封装性, 即子类依赖于其超类中特定功能的实现细节, 超类的实现有可能会随着发行版本的不同而有所变化, 如果真的发生了则子类可能会遭到破坏; 以及子类覆盖了超类的方法以满足某些功能需要, 当超类增加新方法时子类调用则可能不满足功能, 导致异常; 以及超类后续添加了与子类相同方法签名但不同返回类型的方法时, 会导致子类无法编译; 所以如果不是专用的继承则可能导致种种问题, 幸运的是可以使用复合替换继承来避免这些问题, 在新的类中增加一个私有域引用现有类的一个实例, 新类中的每个实例方法都可以调用被包含的现有实例中对应的方法并返回结果, 新的类也被称为包装类, 也正是 Decorator 模式  
继承的功能非常强大, 但是也存在诸多问题, 因为其违背了封装原则, 只有子类和超类之间确实存在子类型关系时, 使用继承才是恰当的; 当子类和超类不在同一个包中时, 并且超类不是专门设计用于继承的, 则建议使用包装模式替代继承, 并且包装类更加稳定和强大

#### 要么为继承而设计, 并提供说明文档, 要么禁止继承
对于专门为了继承而设计的类的文档必须精确的描述覆盖每个方法所带来的影响, 即该类必须有文档说明它可覆盖的方法的自用性; 对于每个公有的或受保护的方法或者构造器, 它的文档必须指明该方法或者构造器的调用了哪些可覆盖的方法, 是以什么顺序调用的, 每个的调用结果又是如何影响后续的处理过程的  
关于程序文档有句格言: 好的 API 文档应该描述一个给定的方法做了什么工作, 而不是描述它如何做到的; 为了继承而进行的设计不仅仅设涉及自用模式的为胆囊设计, 为了使程序员能够编写出更加有效的子类, 而无需承受不必要的痛苦, 类必须通过某种形式提供适当的钩子, 以便能够进入它的内部工作流中, 这种形式可以是精心选择的受保护的方法  
为了允许继承, 类还必须遵守其他一些约束; 构造器决不能调用可被覆盖的方法, 无论是直接调用的还是间接调用的, 如果违反了此规则可能导致程序失败; 超类的构造器是在子类的构造器之前运行的, 所以子类中覆盖版本的方法将会在子类的构造器运行之前就先被调用, 如果该覆盖版本的方法依赖于子类构造器的任何初始化工作, 该方法将不会如预期般执行
```
public class Super {
    // Broken - constructor invokes an overridable method
    public Super() {
        overrideMe();
    }

    public void overrideMe() {}
}

public final class Sub extends Super {
    private final Date date;

    Sub() {
        date = new Date();
    }

    // overriding method invoked by superclass constructor
    @override
    public void overrideMe() {
        System.out.println(date);
    }

    public static void main(String[]  args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```
在为了继承而设计类的时候, Cloneable 和 Serializable 接口出现了特殊的困难; 如果类是为了继承而被设计的, 无论实现这个其中的那个接口通常都不是好主意, 因为它们把一些实质性的覆盖转嫁到了扩展这个类的程序员身上  
如果决定在一个为了继承而设计的类中实现 Cloneable 或者 Serializable 接口, 就应该意识到因为 clone 和 readObject 方法在行为上非常类似于构造器, 所以类似的限制规则也是适用的: 无论是 clone 还是 readObject 都不可以调用可覆盖的方法, 不管是以直接还是间接的方式; 对于 readObject 方法覆盖版本的方式将在子类的状态被反序列化 (deseriable) 之前先被运行, 对于 clone 方法覆盖版本的方式则是在子类的 clone 方法还有机会修正正在被克隆对象的状态之前先被运行, 无论哪种情形都不可避免导致程序失败; 如果决定在一个为继承而设计的类中实现 Serializable, 并且该类有一个 readResolve 或者 writeReplace 方法, 就必须使得 readResolve 或者 writeReplace 成为受保护的方法而不是私有方法, 如果这些方法是私有的, 那么子类将会忽略掉这两个方法, 这正是 "为了继承, 而把实现细节变成一个类的 API 的一部分" 的情景  
对于为了继承而设计的类, 对于这个类会有一些实质性的限制是容易做到的; 但是对于普通的具体类如何避免被子类化, 一种方法是将这个类声明为 final 的, 另一种方法是把所有的构造器都变成私有的或者包级私有的, 并增加一些公有的静态工厂来代替构造器, 使用包装类扩展功能; 如果允许普通的具体类被子类化, 一种合理的办法是确保这个类不会调用它的任何可覆盖的方法, 即完全消除这个类中可覆盖方法的自用特性, 这样就可以 "安全的子类化" 类, 覆盖方法将永远不会影响其他任何方法的行为; 手动消除类中可覆盖方法的自用特性, 而不改变它的特性的方式是, 将每个可覆盖方法的代码迁移到一个私有的 "辅助方法" 中, 用 "直接调用可覆盖方法的私有辅助方法" 来代替 "可覆盖方法的每一个自用调用"

#### 第 18 条: 接口优于抽象类
Java 语言提供了两种机制用来定义允许多个实现的类型: 接口和抽象类; 这两种机制之间的区别在于, 抽象类允许包含某些方法的实现, 但是接口则不允许 (但 JDK8 支持了 default 方法); 一个更为重要的区别在于, 为了实现抽象类定义的类型, 类必须成为抽象类的一个子类, 而任何一个类只要它定义了所有必要的方法, 并且遵守通用约定, 它就被允许实现一个接口, 并且不用管这个类是处于类层次的那个位置; 因为 Java 只允许但继承, 所以抽象类作为类型定义收到了极大的限制
- 现有的类可以很容易被更新, 以实现新的接口
如果这些方法不存在, 所需要做的就是增加必要的方法, 然后在类的声明中增加一个 implements 子句; 而更新现有的类扩展新的抽象类, 则可能出现不能扩展或者扩展会破坏继承结构等问题
- 接口是定义 minin (混合类型) 的理想选择
mixin 是指这样的类型: 类除了实现它的 "基本类型 (primary type)" 之外, 还可以实现这个 mixin 类型, 以表明它提供了某些可供选择的行为; 接口适合做这样的事而抽象类不适合
- 接口允许构造非层次结构的类型框架
类型层次对于组织某些事物是非常合适的, 但是其他有些事物并不能被整齐地组织成一个严格的层次结构

虽然接口不允许包含方法的实现, 但是使用接口来定义类型并不妨碍提供实现上的帮助; 通过对导出每个重要接口都提供一个抽象的骨架实现类, 把接口和抽象类的优点结合起来; 接口的作用仍然是定义类型, 但是骨架实现类接管了所有与接口实现相关的工作; 按照惯例, 骨架实现类称为 AbstractInterface, Interface 是指实现的接口的名字 (例如 AbstractCollection, AbstractSet, AbstractList, AbstractMap), 骨架实现类设计得当可以其他人很容易提供自己的接口实现  
骨架实现的美妙之处在于, 它们为抽象类提供了实现上的帮助, 但又不强加 "抽象类被用作类型定义" 所特有的严格限制; 对于接口的大多数实现来讲, 扩展骨架实现类是个很显然的选择, 但不是必需的; 如果预置类无法扩展骨架实现类, 这个类始终可以手工实现这个接口; 此外骨架实现类仍然能够有助于接口的实现, 实现了这个接口的类可以把对这个接口方法的调用转发到一个内部私有类的实例上, 这个内部私有类扩展了骨架实现类, 这种方法方法被称为模拟多重继承; 编写骨架实现类相对比较简单, 必须认真研究接口, 并确定哪些方法是最为基本的, 其他方法则可以根据它们来实现, 这些基本方法将成为骨架实现类中的抽象方法; 因为骨架实现类是为了继承的目的设计的, 所以需要遵循第 17 条中的原则  
使用抽象类来定义多个实现的类型, 与使用接口相比有一个明显的优势: 抽象类的演变比接口的演变要容易的多, 在后续的发行版本中, 始终都可以增加具体的方法, 对于接口则行不通; 想要在公有接口中增加方法, 而不破坏这个接口的所有实现类是不可能的, 所以必须保证接口初次设计就是正确的  
简而言之, 接口通常是定义允许多个实现的类型的最佳途径, 但这条规则有个例外, 即当演变的容易性比灵活性和功能更为重要的时候, 应该使用抽象类来定义类型, 但前提是必须理解并可以接受其局限性

#### 第 19 条: 接口只用于定义类型
当类实现接口时, 接口就充当可以引用这个类的实例的类型, 因此类实现了接口就表明客户端可以对这个类的实例实施某些动作, 为了其他任何目的而定义接口是不恰当的  
常量接口模式是对接口的不良使用, 类在内部使用某些常量属于实现细节, 实现常量接口会导致把这样的实现细节泄露到该类的导出 API 中; 如果导出常量可以考虑以下几种方案
- 如果这些常量与某个现有的类或者接口紧密相关, 就应该把这些常量添加到这个类或者接口中
- 如果这些常量可以被看做枚举成员, 就应该使用枚举
- 否则就应该使用不可实例化工具类

简而言之, 接口应该只被用来定义类型, 而不应该被用来导出常量

#### 第 20 条: 类层次优于标签类
```
// Tagged class
class Figure {
    enum Shape {RECTANGE, CIRCLE};

    final Shape shape;

    double length;
    double width;

    double radius;

    Figure(double length, double width) {
        shape = Shape.RECTANGE;
        this.length = length;
        this.width = width;
    }

    Figure(double radius) {
        shape = shape.CIRCLE;
        this.radius = radius;
    }

    double area() {
        switch(shape) {
            case RECTANGE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError();
        }
    }
}
```
这样的标签类充斥着样板代码, 包括枚举声明, 标签域以及条件语句, 破坏了可读性, 并且内存占用增加了; 标签类过于冗长, 容易出错且效率低下; 在 Java 应该避免标签类而使用类层次处理
```
abstract class Figure {
    abstract double area();
}

class Rectange extends Figure {
    final double length;
    final double width;

    Rectange(double length, double width) {
        this.length = length;
        this.width = width;
    }

    double area() {
        return length * width;
    }
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    double area() {
        return Math.PI * (radius * radius);
    }
}
```
类层次的另一种好处是, 可以用来反映类型之间本质上的层次关系, 有助于增强灵活性, 并进行更好的编译时类型检查

#### 第 21 条: 用函数对象表示策略
有些语言支持函数指针 (function pointer), 代理(delegate), lambda 表达式 (lambda expression), 或者支持类似的机制, 允许程序把 "调用特殊函数的能力" 存储起来并传递这种能力  
Java 用对象引用实现了类似 C 语言函数指针的功能; 调用对象上的方法通常是执行该对象上的某项操作, 即可以定义这样一种对象, 它的方法执行其他对象 (这些对象被显式传递给这些方法) 上的操作, 如果一个类仅仅导出这样的一个方法, 它的实例实际上就等同于一个指向该方法的指针, 这样的实例被称为函数对象 (function object)
```
class StringLengthComparator {
    public int compare(String s1, String s2) {
        return s1..length() - s2.length();
    }
}
```
指向 StringLengthComparator 对象的引用可以被当作一个指向该比较器的 "函数指针 (function pointer)", 可以在任意一对字符串上被调用; 换句话说, StringLengthComparator 的实例是用于字符串比较操作的具体策略 (concrete strategy); 作为具体策略类, StringLengthComparator 的实例是无状态的, 它没有域, 所以所有实例都是等价的, 因此作为一个 Singleton 是非常合适的; 当以匿名类使用时, 每次执行都会创建一个新的实例, 如果有被重复使用可以将函数对象存储到一个私有的静态 final 域里, 这样做的还可以为其起一个有意义的名字  
因为策略接口被用作所有具体策略实例的类型, 所以不需要为了导出具体策略而把具体策略类做成公有的, 相反 "宿主类" 还可以导出公有的静态域 (或者静态工厂方法), 其类型为策略接口, 具体的策略类可以是宿主类的私有嵌套类; String 类利用这种模式, 通过 CASE_INSENSITIVE_ORDER 域导出了一个不区分大小写的字符串比较器
```
class Host {

    private static class StrLenCmp implements Comparator<String>, Serializable {
          public int compare(String s1, String s2) {
              return s1..length() - s2.length();
        }
    }

    // return comperator is serializable
    public static final Comperator<String> STRING_LENGTH_COMPARATOR = new StrLenCmp();
}
```
函数指针的主要作用是实现策略模式, 为了在 Java 中实现这种模式, 要声明一个接口来表示该策略, 并且为每个具体的策略声明一个实现了该接口的类; 当一个具体策略只被使用一次, 通常使用匿名类来声明和实例化这个具体策略类, 当一个具体策略类是设计用来被重复使用的时候, 它的类通常就要被实现为私有的静态成员类, 并通过公有的静态 final 域导出, 其类型为该策略接口

#### 第 22 条: 优先考虑静态成员类
嵌套类 (nested class) 是被定义在另一个类的内部的类; 嵌套类存在的目的应该知识为它的外围类 (enclosing class) 提供服务; 如果嵌套类将来可能会用于其他的某个环境中, 则就应该是顶层类; 嵌套类有四种: 静态成员类 (static member class), 非静态成员类 (nonstatic member class), 匿名类 (anonymous class) 和局部类 (local class); 除了第一种, 其余三种都被称为是内部类 (innser class)  
静态成员类是最简单的一种嵌套类, 最好将其看作是普通的类, 只是碰巧被声明在另一类的内部而已, 它可以访问外围类的所有成员, 包括那些明为私有的成员; 静态成员类是外围类的一个静态成员, 与其他的静态成员一样, 也遵守同样的可访问性规则; 静态成员类的一种常见的用法是作为公有的辅助类, 仅当与它的外部类一起使用时才有意义  
从语法上讲, 静态成员类和非静态成员类之间唯一的区别是, 静态成员类的声明中包含修饰符 static, 尽管语法非常相似, 但是这两种嵌套类有很大的不同; 非静态成员类的每个实例都隐含着与外围类的一个外围实例相关联, 在非静态成员类的实例内部方法可以调用外围实例上的方法, 或者利用修饰过的 this 构造获得外围实例的引用; 如果嵌套类的实例可以在外围类的实例之外独立存在, 这个嵌套类就必须是静态成员类, 在没有外围实例的情况下, 想要创建非静态成员类的实例是不可能的; 非静态成员类的一种常见用法是定义一个 Adapter, 它允许外部类的实例被看做是另一个不相关的类的实例, 例如 Set, List 集合接口实现使用非静态成员类来实现它们的迭代器  
如果声明成员类不要求访问外围实例, 就要始终把 static 修饰器放在声明中, 使之成为静态成员类, 以减少外围类和非静态成员类之间的关联引用, 避免导致外围实例在符合垃圾回收时仍被保留的可能  
匿名类不同于其他嵌套类, 匿名类没有名字, 也不是外围类的成员, 不与其他成员一起声明, 只在使用的同时被声明和实例化; 当且仅当匿名类出现在非静态的环境中时, 它才有外围实例, 即使匿名类出现在静态环境中也不可能拥有任何静态成员; 匿名类的一种常见用法是动态地创建函数对象 (见第 21 条)  
局部类是四种嵌套类中用的最少的类, 在任何 "可以声明局部变量" 的地方, 都可以声明局部类; 与成员类一样, 局部类也有名字, 可以重用; 与匿名类一样, 只有当局部类是在非静态环境中定义的时候才有外围类, 它们也不可能包含静态成员  
如果一个嵌套类需要在当个方法之外是可见的, 或者它太长了不适合放在方法内部, 就应该使用成员类; 如果成员类的每个实例都需要一个指向其外围实例的引用, 就要把成员类做成非静态的; 否则应该做成静态的; 假设这个嵌套类属于一个方法的内部, 只需要在一个地方创建实例, 并且已经有了一个预置的类型可以说明这个类的特征, 就要把它做成匿名类
