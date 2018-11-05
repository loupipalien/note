### 方法

#### 第 38 条: 检查参数的有效性
绝大多数方法和构造器对于传递给它们的参数值都会有某些限制, 例如索引值必须是非负数, 对象引用不能为 null 等等; 这些应该在文档中清楚的指明, 并且在方法体的开头处检查参数, 以强制施加这些限制; 这是 "应该在发生错误之后尽快检测出错误" 这一普遍原则的一个具体情形; 如果不能做到这一点, 检测到错误的可能性就比较小了, 即使检测到了错误, 也较难确定错误的根源   
不要从本条目内容得出这样的结论: 对参数的任何限制都是好事; 相反, 在设计方法时, 应该使它们尽可能的通用, 并符合实际的需要; 假如方法对于它能接受的所有参数值能够完成合理的工作, 对参数的限制越少越好

#### 第 39 条: 必要时进行保护性拷贝
使 Java 使用起来如此舒适的一个因素在于它是一门安全的语言 (safe language), 并不像 C 或者 C++ 那样把内存当作一个巨大的数组来看待, 所以对于内存破坏的错误都可以免疫; 但即使在安全的语言中, 如果不采取一点措施, 还是无法与其他类隔离开来; 假设客户端尽其所能的来破坏, 或者对 API 的误用所导致的不可预期的情况, 都只好由类来处理; 编写一些面对客户的不良行为时仍能保持健壮的类, 这是非常值得投入的事情  
没有对象的帮助, 另一个类不可能修改对象的内部状态, 但是对象很容易在无意识的情况下提供这种帮助; 考虑以下的类, 声称可以表示一段不可变的时间周期
```
// Broken "immutable" time period class
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {

        this.start = start;
        this.end = end;
    }

    public Date start() {
        retuen start;
    }

    public Date end() {
       return end;
    }
    ...
}
```
乍一看这个类是不可变的, 然而因为 Date 类本身是可变的, 因此很容易违反这个条件; 为了保护 Period 实例的内部信息避免受到这种攻击, 首先对于构造器的每个可变参数进行保护性拷贝 (defensive copy) 是必要的, 并且使用备份对象作为 Period 实例的组件, 而不适用原始的对象
```
// Required constructor - makes defensive copies of parameters
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime);

    if (start.comparaTo(end) > 0) {
        throw new IllegalArgumentException(start + "after" + end);
    }
}
```
用了新的构造器后, 外部对于传递给构造器的 Date 对象的修改不会影响到 Period 实例; 注意, 保护性拷贝是在检查参数的有效性 (见第 38 条) 之前进行的, 并且有效性检查是针对拷贝州的对象, 而不是针对原始的对象; 这样看起来有点不太自然, 但却是有必要的; 这样做可以避免在 "危险阶段" 期间从另一个线程改变类的参数, 这里的危险阶段是指从检查参数开始直到拷贝参数之间的时间段 (在计算机安全社区中, 被称作 Time-Of-Check/Time-Of-Use 或者 TOCTOU 攻击)  
同时也注意, 这里没有使用 Date 的 clone 方法来进行保护性拷贝, 因为 Date 是非 final 的, 不能保证 clone 方法一定返回类为 java.util.Date 的对象: 有可能返回专门处于恶意目的而设计的不可信子类的实例; 例如, 这样的子类可以在每个实例被创建的时候, 把指向该实例的引用记录到一个私有的静态列表中, 并且允许攻击者访问列表, 这将使得攻击者可以控制所有实例; 为了阻止这种攻击, 对于类型参数可以被不可信任子类化的参数, 不要使用 clone 方法进行保护性拷贝  
虽然构造方法成功避免了攻击, 但实例域的访问方法被外部访问, 仍有可能改变实例域的值; 为了防止这种攻击, 需要修改实例域的访问方法
```
// Required accessors - make defensive copies of internal fields
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```
采用了新的构造器和新的访问方法后, Period 实例域被封装在对象的内部; 访问方法与构造器不同, 它们在进行保护性拷贝时可以使用 clone 方法, 因为可以保证 Period 内部的 Date 对象是 java.util.Date, 而不可能是其他的某个潜在的不可信子类; 对象的内部组件在返回给客户端之前, 应该考虑是否进行保护性拷贝; 对于长度非零的数组总是可变的, 因此把内存不数组返回给客户端之前, 应该进行保护性拷贝, 另外一种解决方案是返回该数组的不可变视图, 这两种方法见第 13 条  
以上启示在于, 只要有可能都应该使用不可变的对象作为对象内部组件, 这样就不必再进行保护性拷贝; Period 类可将 Date 域变成 long 类型表示, 这样就可以避免保护性拷贝; 如果类所包含的方法或者构造器的调用需要移交对象的控制权, 这个类就无法让自身抵御恶意的客户端, 只能靠类与客户端之间的信任  
简而言之, 如果类具有从客户端得到或着返回到客户端的可变组件, 类就必须保护性拷贝这些组件, 如果拷贝的成本受限制, 并且类信任客户端不会不恰当的修改组件, 就可以在文档中指明客户端的指责是不得修改受影响的组件, 依此来代替保护性拷贝

#### 第 40 条: 谨慎设计方法签名
- 谨慎的的选择方法的名称
方法名称应该时钟遵循标准的命名习惯; 首要目标是选择容易理解的, 并且与同一个包中的其他名称风格一致的名称; 第二个目标应该是选择与大众认可的相一致的名称
- 不要过于追求提供便利的方法
每个方法都应该尽其所能, 方法太多会使类难以学习, 使用, 测试, 维护; 对于接口和类所支持的每个动作, 都提供一个功能齐全的方法, 只有当一项操作经常被用到的时候, 才考虑为它提供快捷方式, 如果不能确定还是不提供的好
- 避免过长的参数列表
目标是不多于四个参数, 如果编写的许多方法都超过了这个限制, 那么 API 就会变得不好使用, 需要经常的查看文档; 相同类型的长参数序列格外有害, 会难以记住参数的顺序而导致出错

有三种方法可以缩短过长的参数列表; 第一种是把方法分解成多个方法, 每个方法只需要这些参数的一些子集, 如果这样做不小心导致了方法过多, 可以通过提升方法的正交性来减少方法的数目; 第二个方法时创建辅助类, 用来保存参数的分组, 这些辅助类一般为静态成员类 (见第 22 条), 如果一个频繁出现的参数序列可以被看作是代表了某个独特的实体, 则建议使用这种方法; 第三种方法是从对象构建到方法调用都采用 Builder 模式 (见第 2 条), 如果方法带有多个参数, 尤其是当它们中有些是可选的使用, 最好定义一个对象来表示所有参数, 并允许客户端在这个对象上进行多次 "setter" 调用, 每次调用都设置一个参数, 或者设置一个较小的相关的集合  
对于参数类型, 要优先使用接口而不是类 (见第 52 条), 只要有适当的接口可用来定义参数, 就优先使用这个接口而不是使用实现该接口的类; 如果使用的是类而不是接口, 则限制了客户端只能传入特定的实现, 如果碰巧输入的数据是以其他形式存在的, 就会导致不必要的, 可能非常昂贵的拷贝操作; 对于 boolean 参数, 要优先使用两个元素的枚举类型, 它是代码更易于阅读, 尤其是在使用支持自动完成功能的 IDE 的时候, 也使以后更易于添加更多的选项; 例如可能会有一个  Thermometer 类型, 带有一个静态工厂方法, 而这个静态工厂方法的签名需要传入这个枚举的值:
```
public enum ThermometerScale {FAHRENNEIT, CELSIUS}
```
Thermometer.newInstance(ThermometerScale.CELSIUS) 不仅比 Thermometer.newInstance(true) 更易读, 而且还可以在未来的发行版本中将 KELVIN 添加到 ThermometerScale 中

#### 第 41 条: 慎用重载
```
// Broken - What does this program print ?
public class CollectionClassifier {

    public static String classify(Set<?> set) {
      return "Set";
    }

    public static String classify(List<?> list) {
      return "List";
    }
    public static String classify(Collection<?> collection) {
      return "Unknow Collection";
    }

    public static void main(String[] args) {
      Collection<?>[] collections = {
              new HashSet<String>(),
              new ArrayList<String>(),
              new HashMap<String,String>().values()};

      for (Collection<?> collection : collections) {
          System.out.println(classify(collection));
      }
    }
}
```
以上代码试图根据一个集合是 Set, List, 还是其他集合类型来对其分类; 可能期望以上代码打印出 " Set, Lis, Unknow Collection", 但实际上是打印了三次 "Unknow collection"; 由于 classify 方法被重载了, 而要调用哪个重载 (overloading) 方法是在编译时做出决定的, 对于 for 循环中的全部迭代参数编译都是相同的 Collection<?> 类型, 即使每次迭代的运行时类型是不同的, 但这并不影响对重载方法的选择; 即对于重载方法的选择是静态的, 对于被覆盖的方法选择是动态的, 因为覆盖机制是规范, 而重载机制是例外, 所以覆盖机制满足了人们对方法调用行为的期望, 而重载机制很容易使这些期望落空; 如果 API 的普通用户根本不知道 "对于一组给定的参数, 其中的哪一个重载方法会被调用", 那么使用这样的 API 就很可能导致错误, 因此应该避免胡乱的使用重载机制  
对于使用重载机制的保守策略是: 永远不要导出两个具有相同参数数目的重载方法, 如果方法使用可变参数, 保守策略是根本不要重载; 可以参考 ObjectOutputStream 类对于 write 方法参数是不同类型的变形, 可以始终给方法起不同的名字, 而不使用重载机制; 对于构造器可能不能使用这样的方法, 一个类的多个构造器总是重载的, 在许多情况下, 可以选择导出静态工厂, 而不是构造器(见第 1 条)  
在 Java 1.5 发行版本之前, 所有的基本类型都根本不同于所有的引用类型, 但是当自动装箱拆箱出现后, 就不再如此, 它会导致真正的麻烦, 例如以下程序
```
publi class SetList {
     public static void main(String[] args) {
     Set<Integer> set = new TreeSet<>();
     List<Integer> list = new ArrayList<>();

      for (int i = -3; i < 3; i++) {
          set.add(i);
          list.add(i);
      }
      for (int i = 0; i < 3; i++) {
          set.remove(i);
          list.remove(i);
      }

      System.out.println(set + " " + list);
    }
}
```
可能预期输出是 "[-3, -2, -1] [-3, -2, -1]", 但实际上输出的是 "[-3, -2, -1] [-2, 0, 2]"; 这是因为 Set 调用的方法是 remove(Object o), List 调用的方法是 remove(int i); JDK8 中 List 的 remove 方法调用正确, 重载方法在编译时静态确认, 装箱方法在运行时才会被调用  
简而言之, "能够重载的方法" 并不意味着 "应该重载方法"; 一般情况下, 对于多个具有相同参数数目的方法来说, 应该尽量避免重载方法; 在某些情况下不能保证如此, 那么应该避免同一组参数只需经过类型转换就可以被传递给不同的重载方; 在某些情况下不能保证如此, 则应该保证当传递同样的参数时, 所有重载方法的行为必须一致; 如果不能做到以上, 对于被重载的方法或者构造器的使用就容易被混淆

#### 第 42 条: 慎用可变参数
Java 1.5 发行版本中增加了可变参数, 一般称作 variable arity method; 可变参数方法接受 0 或多个指定类型的参数, 可变参数的机制是通过先创建一个数组, 数组的大小为在调用位置所传递的参数个数, 然后将参数值传递到数组中, 然后经数组传递给方法
```
// Sample use of varargs
static int sum(int... args) {
    int sum = 0;
    for (int arg : args) {
        sum += arg;
    }
    return sum;
}
```
有时候需要编写 1 个或者多个某种类型的参数方法, 而不是 0 个或多个
```
// The WRONG way to use varargs to pass one or more arguments
static int min(int... args) {
    if (args.length == 0) {
        throw new IllegalArgumentException("Too few arguments.");
    }
    int min = args[0];
    for (int i = 0; i < args.length; i++) {
        if (args[i] < min) {
            min = args[i];
        }
    }
    return min;
}
```
这样的代码虽可行, 但当客户端没有传递参数时, 就会在运行时而不是编译时失败; 另一个问题是这段代码包含参数校验, 并不美观; 以下是另一种声明, 解决了这种不足
```
static int min(int first firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs) {
        if (arg < min) {
            min = arg;
        }
    }
    return min;
}
```
正如以上所见, 当需要一个让方法带有不定数量的参数时, 可变参数就变得非常有效; 可变参数是为 printf 而设计的, 在 Java 1.5 发行版本中发布, 为了核心的反射机制 (见第 53 条), 在该发行版本中被改造成利用可变参数, printf 和反射机制都从可变参数中极大受益  
在 Java 1.5 版本发行之前, 打印数组的常用做法是: System.out.println(Arrays.asList(array)); 但这种做法只有在对象引用类型的数组上才有用, 如果不小心在基本类型的数组上尝试这样做, 程序将无法编译; 但在 Java 1.5 中 Arrays.asList 改造成可变参数的办法, 现在基本类型数组也可以通过编译, 并且没有错误和警告; 但 Arrays.asList 方法现在 "增强" 为使用可变参数, 如果传入基本类型的数组, 会将基本类型数组的引用作为列表的元素, 打印时产生迷惑的结果; 但在 Java 1.5 发行版本中也设计了 Arrays.toString() 的方法, 专门为了将任何类型的数组转变成字符串设计的  
在重视性能的情况下, 使用可变参数机制要特别小心, 可变参数方法的每次调用都会导致一次数组分配和初始化; 如果无法承受这一成本, 但又需要可变参数的灵活性, 则有一种方法可以如愿以偿; 假设对某个方法的 95% 的调用会有 3 个或者更少的参数, 就应该声明该方法的 5 个重载, 每个重载方法带有 0 到 3 个普通参数, 当参数的数目超过 3 个时, 就使用一个可变参数的方法
```
public void foo() {...}
public void foo(int a1) {...}
public void foo(int a1, int a2) {...}
public void foo(int a1, int a2, int a3) {...}
public void foo(int a1, int a2, int a3, int... rest) {...}
```
EnumSet 类对它的静态工厂使用这种方法, 最大幅度的减少创建枚举集合的成本

#### 第 43 条: 放回零长度的数据或集合, 而不是 null
简而言之, 返回类型为数组或者集合的方法没理由返回 null, 而不是返回一个零长度的数组或者集合

#### 第 44 条: 为所有导出的 API 元素编写文档注释
TODO
