### 枚举与注解
JDK1.5 增加了两个新的引用类型家族, 一种新的类称为枚举类型 (enum type), 一种新的接口称作注解类型 (annotation  type)

#### 第 30 条: 用 enum 代替 int 常量
枚举类型是指由一组固定的常量组成合法值的类型; 在没有引入枚举类型之前, 表示枚举类型常用模式是声明一组具名 int 或 String 的常量, 称为 int 枚举模式或者 String 枚举模式
```
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final String ORANGE_NAVEL = "0";
public static final String ORANGE_TEMPLE = "1";
public static final String ORANGE_BLOOD = "2";
```
但以上方法存在这诸多不足, 在类型安全性和使用方便性方面没有任何帮助, int 枚举模式打印无可读性, String 枚举类型比较效率差; 枚举类型可以避免以上缺点, 并提供一些额外的好处
```
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
public enum Apple {NAVEL, TEMPLE, BLOOD}
```
Java 的枚举类型本质上是 int 值, 背后的基本想法非常简单: 就是通过公有的静态 final 域为每个常量导出实例的类; 因为没有可以访问的构造器, 枚举类型是真正的 final; 枚举类型是实例受控的, 是单例的泛型化, 本质上是单元素的枚举; 枚举还允许添加任意的方法和域, 并实现任意的接口, 提供了所有 Object 方法的高级实现, 实现了 Comparable (见第 12 条) 和 Serializable 接口 (见第 11 章), 并针对枚举类型的可任意改变性设计了序列化方式; 提供一个 values 方法按照声明返回它的值的数组, toString 方法返回每个枚举值的声明名称
```
public enum Operation {
    PLUS, MINUS, TIMES, DIVIVE;

    // Do the arithmetic op representated by this constant
    double apply(double x, double y) {
        switch(this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVIVE: return x / y;
        }
        throw new AssertionError("Unknown Operation:" + this);
    }
}
```
这段代码可行但是不优雅; 如果没有 throw 语句就不能编译, 虽然从技术的角度看来是可行的, 但是 throw 语句是不可能执行到的; 而且这段代码很脆弱, 如果添加了新的枚举常量, 却忘记给 switch 添加相应的条件枚举仍然能编译, 运行时就会失败; 有一种方法可以将不同的行为与每个枚举常量关联起来: 在枚举类型中声明一个抽象的 apply 方法, 并在特定于常量的类主体 (constant-specific class body) 中用具体的方法覆盖, 这种方法被称作特定于常量的方法实现 (constant-specific method implmentation)
```
public enum Operation {
    PLUS {double apply(double x, double y) {return x + y;}},
    MINUS {double apply(double x, double y) {return x - y;}},
    TIMES {double apply(double x, double y) {return x * y;}},
    DIVIVE {double apply(double x, double y) {return x / y;}};

    double abstract apply(double x, double y);
}
```
此外, 枚举类型还有一个自动产生的 valueOf(String) 方法, 它将常量的名字转变成常量本身; 如果在枚举类型中覆盖了 toString, 要考虑编写一个 fromString 方法, 将定制的字符串方法变回相应的枚举  
一般来说, 当需要一组固定常量的时候, 考虑使用枚举; 与 int 常量相比, 枚举类型的优势是不言而喻的, 枚举也易读的多, 也更加安全以及强大

#### 第 31 条: 用实例域代替序数
许多枚举天生就与一个单独的 int 值相关联, 所有的枚举都有一个 ordinal 方法, 它返回每个枚举常量在类型中的数字位置, 可以尝试着从序数中得到关联的 int 值
```
// Abuse of ordinal to derive an associated value - DON'T DO THIS
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() {
        return ordianl() + 1;
    }
}
```
虽然这个枚举不错, 但是维护起来就像一场噩梦, 如果常量进行重新排序, numberOfMusicians 方法就会遭到破坏; 幸运的是有一种简单的方法可以解决这个问题: 永远不要根据枚举的序数导出与它关联的值, 而是要将它保存在一个实例中
```
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), SEXTET(6), SEPTET(7), OCTET(8), NONET(9), DECTET(10);

    public final int numberOfMusicians;

    Ensemble(int size) {
        this.numberOfMusicians = size;
    }

    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
```
Enum 的 ordinal 方法注释中写道: 大多数程序员将不会使用到这个方法, 它是设计为像 EnumSet 和 EnumMap 这种基于枚举的通用数据结构的; 所以除非编写的是这种数据结构的代码, 否则最好完全避免使用 ordinal 方法

#### 用 EnumSet 代替位域
如果一个枚举类型主要用在集合中, 一般就使用 int 枚举模式, 将 2 的不同倍数赋予每个常量
```
// Bit field enumeration constants - OBSOLETE
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // parameter is bitwise OR of zero or more STYLE_ constants
    public void applyStyles(int styles) {
        ...
    }
}
```
位域表示法也允许利用位操作, 有效的执行像 union 和 intersection 这样集合操作, 但位域有着 int 枚举常量所有的缺点; 其实有更好的替代方法, java.util 包提供了 EnumSet 类来有效地表示从单个枚举类型中提取多个值的集合; 这个类实现了 Set 接口, 提供了丰富的功能, 类型安全性以及可以从任何其他 Set 实现中得到的互用性; 但是在内部具体的实现上, 每个 EnumSet 内容都表示为位矢量; 如果底层的枚举类型有 64 个或者更少的元素 (大多如此), 整个 EnumSet 就是用单个 long 来表示的, 因此它的性能比得上位域的性能
```
// EnumSet - a modern replacement for bit fields
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLIEN, STRIKETHROUGH}

    // Any set could be passed in, but EnumSet is clearly best
    public void applyStyles(Set<Style> styles) {
        ...
    }
}
```
下面是将 EnumSet 实例传递给 applyStyles 方法的客户端代码, EnumSet 提供了丰富的静态工厂来轻松的创建集合
```
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```
总而言之, 正是因为枚举类型要用在集合中, 所以没有道理用位域来表示它, EnumSet 类集位域的简洁和性能优势以及第 30 条中所述的枚举类型优势于一身; 但实际上 EnumSet 有个缺点, 即无法创建不可变的 EnumSet, 但可以用 collections.unmodifiableSet 将其封装, 当简洁性和性能会受到影响

#### 用 EnumMap 代替序数索引
有时可能会见到利用 ordinal 方法来索引数组的代码, 例如以下
```
public class Herb {
    public enum Type {ANNUAL, PERENNIAL, BIENNIAL}

    ptivate final String name;
    private final Type type;

    Herb(String name, Type type) {
        this.name = name;
        this.type = type;
    }

    @override
    public String toString() {
        return name;
    }
}
```
现在假设有一个香草的数组, 表示一座花园中的植物, 如果想要按照类型 (一年生, 两年生, 多年生) 进行组织只有将这些植物列出来, 这样需要构建三个集合, 每种类型一个, 然后变量整个花园, 将每个香草放到相应的集合中
```
// Using ordinal() to index an array - DON'T DO THIS
Herb[] garden = ...;
Set<Herb>[] herbsByType = (Set<Herb>[]) new Set[Herb.Type.values().length];
for(int i = 0; i < herbsByType.length; i++) {
    herbsByType[i] = new HashSet<Herb>();
}
for(Herb h : garden) {
    herbsByType[h.type.ordinal()].add(h);
}
// Print
for(int i = 0; i < herbsByType.length; i++) {
   System.out.printf("%s: %s%n", Herb.Type.values()[i], herbsByType[i])
}
```
这种方法的确可行, 但是隐藏着许多问题, 因为数组和泛型不能兼容, 程序需要进行未受检的转换, 并且不能正确无误地进行编译, 因为数组不知道它的索引代表着什么, 必须手工标注这些索引的输出; 有一种专门用于枚举键且非常快速的 Map 可以代替数组充当从枚举到值的映射, 即 java.util,EnumMap
```
// Using an EnumMap to associate date with an enum
Map<Herb.Type,Set<Herb>> herbsByType = new EnumMap<>(Herb.Type.class);
for(Herb.Type t : Herb.Type.values()) {
    herbsByType.put(t, new HashSet<>());
}
for(Herb h : garden) {
    herbsByType.get(h.type).add(h);
}
System.out.println(herbsByType);
```
这段程序更简短, 更清楚也更加安全, 运行速度方面可以与使用序数的程序媲美; EnumMap 在运行速度上之所以能与通过序数索引的数组相媲美, 是因为 EnumMap 在内部使用了这种数组, 但是它对程序员隐藏了这种实现细节, 集 Map 的丰富功能和类型安全与数组的快速于一身; 注意 EnumMap 构造器采用键类型的 Class 对象: 这时一个有限制的类型令牌, 它提供了运行时的泛型信息  
总而言之, 最好不要用序数来索引数组, 而要使用 EnumMap

#### 第 34 条: 用接口模拟可伸缩的枚举
第 30 条中的 Operation 类型表示一个简单的计算器中的函数, 有时候要尽可能地让 API 用户提供它们自己的操作, 这样可以有效的扩展 API 所提供的操作集; 有一种很好的方法可以利用枚举来实现可伸缩的效果, 由于枚举类型可以实现接口, 基本想法就是利用这一事实
```
// Emulated extensible enum using an interface
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implments Operation {
  PLUS("+") {double apply(double x, double y) {return x + y;}},
  MINUS("-") {double apply(double x, double y) {return x - y;}},
  TIMES("*") {double apply(double x, double y) {return x * y;}},
  DIVIVE("/") {double apply(double x, double y) {return x / y;}};

  private final String symbol;
  BasicOperation(String symbol) {
      this.symbol = symbol;
  }

  @override
  public String toString() {
      return symbol;
  }
}
```
虽然枚举类型不可扩展, 但是接口是可扩展的; 还可以定义另一个枚举其他实现 Operation 接口
```
public enum ExtendedOperation implments Operation {
  EXP("+") {double apply(double x, double y) {Math.pow(x, y);}},
  REMINDER("-") {double apply(double x, double y) {return x % y;}};

  private final String symbol;
  ExtendedOperation(String symbol) {
      this.symbol = symbol;
  }

  @override
  public String toString() {
      return symbol;
  }
}
```
只要 API 写成接口类型, 在可以使用基础操作的任何地方都可以使用新的操作
```
public static <T extends Enum<T> & Operation> void test(Class<T> opSet, double x, double y) {
   for(Operation op : opSet.getEnumContants()) {
      System.out.println(op.apply(x, y));
   }
}
public static void test(Collection<? extends Operation>, double x, double y) {
   for(Operation op : opSet) {
      System.out.println(op.apply(x, y));
   }
}
```
总而言之, 虽然无法编写可扩展的枚举类型, 却可以通过编写接口以及实现该接口的基础枚举类型, 对其进行模拟

#### 第 35 条: 注解优先于命名模式
TODO

#### 第 36 条: 坚持使用 Override 注解
这个注解只能用在方法声明中, 它表示被注解的方法声明覆盖了超类型中的一个声明; 坚持使用这个注解, 可以防止许多错误
```
// Can you spot the bug
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
      return b.first = first && b.second = second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> S = new HashSet<Bigram>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }
        System.out.println(s.size());
    }
}
```
运行这个程序, 会发现集合中并不是 26 个元素而是 260 个; 这是因为程序员原本想覆盖 equals 方法 (见第 8 条), 而且还覆盖了 hashCode 方法, 但遗憾的是 equals 方法并没有被覆盖而是被重载： 如果对超类的方法覆盖标注上了 Override 的注解, 在编译时就可以发现报错; 所以应该在想要覆盖超类声明的每个方法声明中使用 Override 注解

#### 第 37 条: 表标记接口定义类型
TODO 
