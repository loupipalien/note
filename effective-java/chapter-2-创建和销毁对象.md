### 创建和销毁对象
何时以及如何创建对象, 何时以及如何避免创建对象, 如何确保它们能够适时地销毁, 以及如何管理对象销毁之前必须进行的各种清理动作

#### 第 1 条: 考虑用静态工厂方法代替构造器
对于类而言, 让客户端获取它自身的一个实例, 最常用的方式是提供一个公有的构造器; 还有一种方法是类提供一个公有的静态工厂方法 (static factory method): 返回类的实例的静态方法; 提供静态工厂方法而不是公有的构造器有几大优势, 同样的也有劣势

##### 优势: 它们有名称
如果构造器的参数本身没有确切的描述正在返回的对象, 那么具有适当名称的静态工厂会更容易使用, 产生的客户端代码也更易阅读; 当类有多个构造器方法时, 方法签名的唯一不同既是构造器参数类型或顺序的不同, 面对这样的 API 用户往往记不住该用哪个构造器, 并且在阅读这些代码时没有参考类的文档也容易迷惑

##### 优势: 不必每次调用它们的时候都创建一个新的对象
这使得不可变类可以使用预先构建好的实例, 或者将构建好的实例缓存起来, 进行重复利用, 从而避免创建不必要的重复对象; 静态工厂方法能够为重复调用返回相同的对象, 这有助于类总能控制在某个时刻哪些实例应该存在; 这种类被称为实例受控的类 (instance-controlled); 编写实例受控制的类有几个原因: 实例受控使得类可以确保它是一个 Singleton (见第 3 条) 或者是不可实例化的 (见第 4 条), 它还使得不可变的类 (见第 15 条) 可以确保不会存在两个相等的实例, 即当且仅当 a==b 的时候才有 a.equals(b) 为 true; 如果类保证了这一点, 就可以用 == 操作符来代替 eqauls(Object) 方法, 这样可以提升性能; 枚举 (Enum) 类型 (见第 30 条) 就保证了这一点

##### 优势: 它们可以返回原返回类型的任何子类型的对象
这种灵活性的一种应用是, API 可以返回对象同时又不会使对象的类变成公有的, 以这种方式实现隐藏实现类会使 API 变得非常简洁; 这项技术适用于基于接口的框架 (interface-based framework, 见第 18 条), 在这种框架中接口为静态工厂方法提供了自然返回类型, 由于接口不能有静态方法, 因此按照惯例: 接口 Type 的静态工厂方法被放在一个名为 Types 的不可实例化类 (见第 4 条) 中  
如 Java Collections Framework 的集合接口有 32 中不同的实现, 但几乎所有实现都通过静态工厂方法在一个不可实例化的类中 (java.util.collections) 导出, 所有返回对象的类都是非公有的, 而且这种方式实现方式会小的多; 用户在使用这种静态工厂方法时, 要求用接口来引用被返回的对象, 而不是通过它的实现类来引用被返回的对象, 这是一种良好的习惯 (见第 52 条); 公有的静态工厂方法所返回的对象不仅可以是非公有的, 而且该类该可以随着每次调用发生变化, 这取决于静态工厂方法的参数值, 例如 java.util.EnumSet (见第 32 条) 没有公有构造器只有静态工厂方法, 能依据底层枚举类型的大小返回 RegularSet 实例或 JumboEnumSet 实例  
静态工厂方法返回的对象所属的类, 在编写包含该静态工厂方法的类时可以不必存在; 这种灵活的静态工厂方法构成了服务提供者框架 (Service Provider Framework) 的基础, 服务提供者框架是指这样一个系统: 多个服务提供者实现一个服务, 系统为服务提供者的客户端提供多个实现, 并把它们从多个实现中解耦出来; 服务提供者框架中有三个组件: 服务接口 (Service Interface), 这是提供者实现的; 提供者注册 API (Provider Registration API), 这是系统用来注册实现, 让客户端访问它们的; 服务访问 API (Service Access API), 是客户端来获取服务的实例的; 服务访问 API 一般允许但是不要求客户端指定某种选择提供者条件; 如果没有这样的规定 API 会返回默认实现的一个实例; 服务访问 API 是 "灵活的静态工厂", 它构成了服务提供者框架额基础; 服务提供者框架的第四个组件是可选的: 服务提供者接口 (Service Provider Interface), 这些提供者创建其服务实现的实例, 如果没有服务提供者接口实现就按照类名称注册, 并通过反射方式进行实例化 (见第 53 条); 对 JDBC 来说, Connection 就是它的服务接口, DriverManager.registerDriver 是提供者注册 API, DriverManager.getConnection 是服务访问 API, Driver 是服务提供者接口   
服务提供者框架模式有着无数变体, 以下是一个简单的实现, 包含一个服务提供者接口和一个默认的提供者
```
// Service provider framework sketch

// Service interface
public interface Service {
    ... // Service-specific methods go there
}

// Service provider interface
public interface Provider {
    Service newService();
}

// Noninstantiable class for service Registration and access
public class Services {
    private Services() {} // Prevents instantiation (见第 4 条)

    // Maps service names to services
    private static final Map<String,String> providers = new ConcurrentHashMap<String,Provider>();
    public static final String DEFAULT_PROVIDER_NAME = "<def>";
    // Provider Registration API
    public static void registerDefaultProvider(Provider provider) {
        registerProvider(DEFAULT_PROVIDER_NAME, provider);
    }
    public static final registerProvider(String name, Provider provider) {
        providers.put(name, provider);
    }

    // Service access API
    public static Service newInstance() {
        return newInstance(DEFAULT_PROVIDER_NAME);
    }
    public static Service newInstance(String name) {
        Provider provider = providers.get(name);
        if (provider == null) {
            throw new IllegalArgumentException("No provider registered with name: " + name);
        }
        return p.newService();
    }
}
```

##### 优势: 在创建参数化类型时它们使代码变得更加简洁
在嗲用参数化类的构造器时, 即使类型参数很明显也必须指明, 这通常需要接连两次提供类型参数 (JDK8 中已实现类型推断)
```
Map<String,List<String>> map = new HashMap<String,List<String>>();
```
但是如果提供了静态工厂方法, 编译器就用类型推断帮助找到类型参数, 假设 HashMap 提供了以下静态工厂方法
```
public static <K,V> HashMap<K,V> newInstance {
    return new HashMap<K,V>();
}
```

##### 劣势: 类如果不含有公有的或者受保护的构造器, 就不能被子类化
对于公有的静态工厂返回的非共有类, 同样如此; 如想要将 Conllections Framework 中任何实现类子类化是不可能的; 但这样也许会因祸得福, 因为它鼓励程序员使用复合 (composition) 而不是继承 (见第 16 条)

##### 劣势: 与其他静态方法没有任何区别
在 API 文档中它们没有像构造器爱文档中标识出来, 因此对于只提供了静态工厂方法的类来说, 要查明如何实例化一个类是需要花费一定时间的; 但也可以通过在类或者接口注释中关注静态工厂, 并遵守标准命名习惯弥补这一劣势, 以下是静态工厂方法的一些惯用名称
- valueOf: 不太严格的讲, 该方法返回的实例与它的参数具有相同的值, 这样的静态工厂方法实际上是类型转换方法
- of: valueOf 的一种更简洁形式, 在 EnumSet (见第 32 条) 中使用并流行起来
- getInstance: 返回实例是通过方法的参数来描述的, 但是不能够说与参数具有同样的值
- newInstance: 像 getInstance 一样, 但 newInstance 能够确保返回的每个实例都与所有其他实例不同
- getType: 像 getInstance 一样, 但是在工厂方法处于不同的类中的时候使用; Type 表示 工厂方法所返回的对象类型
- newType: 像 newInstance 一样, 但是在工厂方法处于不同的类中的时候使用; Type 表示 工厂方法所返回的对象类型

简而言之, 静态工厂方法和公有构造器都各有用处, 需要理解它们的各自的长处, 静态工厂通常更加合适, 因此切记第一反应就是提供公有的构造器, 而不考虑静态工厂

#### 第 2 条: 遇到多个构造器参数时要考虑用构建器
静态工厂方法和构造器有个共同的局限性: 都不能很好的扩展到大量的可选参数; 考虑一个类表示包装食品外面 显示的营养成分标签, 其中有几个字段是必填的, 很多字段是选填的  
对于这样的类, 大多数采用的方式是重叠构造器 (telescoping construction) 模式, 在这种模式下, 提供第一个只有必填参数的构造器, 第二个构造器有一个选填的参数, 第三个有两个可选参数, 以此类推, 最后一个参数包含所有可选参数; 在较少参数时, 重叠构造器模式是可行的, 但是当有许多参数的时候, 客户端代码就会很难编写, 并且难以阅读  
第二种方法是 JavaBeans 模式, 在这种模式下调用一个无参构造器来创建对象, 然后调用 setter 方法来设置每个必要的参数以及可选的参数; 但这种方法也有着自身缺点, 因为构造过程被分到了多个调用中, 所以在构造过程中 JavaBean 可能处于不稳定的状态, 试图调用不一致状态的对象将会导致失败; JavaBeans 模式也阻止了把类做成不可变的可能 (见第 15 条), 这就需要付出额外的努力来确保线程安全  
第三种方法是 Builder 模式, 这种方法既能保证像重叠构造器模式那样的安全性, 也能保证像 JavaBeans 模式那么好的可读性; 具体实现是不直接生成想要的对象, 而是让客户端利用所有必填的参数调用构造器 (或静态工厂方法), 得到一个 builder 对象; 然后客户端在 builder 对象上调用类似 setter 的方法来设置每个相关的选填参数, 最后客户端调用无参的 build 方法来生成不可变的对象, 这个 builder 是构建的类的静态成员类; 以下是示例代码
```
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters: initialized to default values
        private int calories = 0;
        peivate int fat = 0;
        private int carbohydrate = 0;
        private int sodium = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this,servings = servings;
        }

        public Builder calories(int calories) {
            this.calories = calories;
            retuen this;
        }

        public Builder fat(int fat) {
            this.fat = fat;
            return fat;
        }

        public Builder carbohydrate(int carbohydrate) {
            this.carbohydrate = carbohydrate;
            return this;
        }

        public Builder sodium(int sodium) {
            this.sodium = sodium;
            return sodium;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }

        public NutritionFacts(Builder builder) {
            servingSize = builder.servingSize;
            servings = builder.servings;
            calories = builder.calories;
            fat = builder.fat;
            carbohydrate = this.carbohydrate;
            sodium = builder.sodium;
        }
    }
}
```
注意 NutritionFacts 是不可变的, 所有的默认参数值都单独放在一个地方, Builder 的 setter 方法返回 Builder 本身, 以便可以把调用链接起来
```
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();
```
builder 像个构造器一般, 可以对其参数强加约束条件, build 方法可以检验这些约束条件; 将参数从 builder 拷贝到对象中之后, 并在对象域而不是 builder 域 (见第 39 条) 中对它进行检验; 如果违反了任何约束条件, build 方法就应该抛出 IllegalArgumentException (见第 60 条), 异常的详细信息应该显示出违反了那个约束条件 (见第 63 条)
