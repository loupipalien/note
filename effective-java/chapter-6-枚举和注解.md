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
    PLUS {double abstract apply(double x, double y) {return x + y;}},
    MINUS {double abstract apply(double x, double y) {return x - y;}},
    TIMES {double abstract apply(double x, double y) {return x * y;}},
    DIVIVE {double abstract apply(double x, double y) {return x / y;}};

    double abstract apply(double x, double y);
}
```
此外, 枚举类型还有一个自动产生的 valueOf(String) 方法, 它将常量的名字转变成常量本身; 如果在枚举类型中覆盖了 toString, 要考虑编写一个 fromString 方法, 将定制的字符串方法变回相应的枚举

#### 
