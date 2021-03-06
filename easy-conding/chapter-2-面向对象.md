### 面向对象
目前主流语言主要分为两大阵营: 面向编程和面向过程; 面向对象编程 (Object-Oriented Programming, OOP) 的抽象, 封装, 继承, 多态的理念使软件大规模化成为可能, OPP 践行了软件工厂的三个目标: 可维护性, 可重用性, 可扩展性

#### OOP 理念
Object 类
- 我是谁: getClass() 说明本质上是谁, toString() 是当前职位的名片
- 我从哪里来: Object() 构造方法是生产对象的基本步骤, clone() 是繁殖对象的另一种方式
- 我到哪里去: finalize() 是在对象销毁时触发的方法
- 世界因我不同: hashCode() 和 equals() 方法就是判断与其他元素是否相同的一组方法
- 与他人协调: wait() 和 notify() 是对象间通信与协作的一组方法

抽象是要找到属性和行为的共性, 属性是行为的基本生产资料, 具有一定的敏感性, 不能直接对外暴露; 封装的主要任务是对属性, 数据, 部分内部敏感行为实现隐藏; 继承允许创建具有逻辑等级结构的类体系, 形成一个继承树, 让软件在业务多变的客观条件下, 某些基础模块可以被直接复用, 间接复用或增强复用, 父类的能力通过这种方式赋予子类; 多态是根据运行时的实际对象类型, 同一个方法产生不同的运行结果, 使同一个行为具有不同的表现形式

#### 初识 Java
JRE (Java Runtime Environment) 即 Java 运行环境, 包括 JVM, 核心类库, 核心配置工具等

#### 类

##### 类的定义
类的定义由访问级别, 类型, 类名, 是否抽象, 是否静态, 泛型标识, 继承或实现关键字, 父类或接口名称等组成; 类的访问级别有 public 和无访问控制符, 类型分 class, interface, enum; Java 类主要由两部分组成: 成员和方法, 方法排序建议: 公有方法 > 保护方法 > 私有方法 > getter/setter 方法

##### 接口和抽象类
| 语法维度 | 抽象类 | 接口 |
| :------------- | :------------- |:------------- |
| 定义关键字 | abstract | interface |
| 子类继承或实现关键字 | extends | implements |
| 方法实现 | 可以有 | 不能有, JDK 8 及以后允许有 default 实现 |
| 方法访问控制符 | 无限制 | 有限制, 默认是 public abstract 类型 |
| 属性访问控制符 | 无限制 | 有限制, 默认是 public static final 类型 |
| 静态方法 | 可以有 | 不能有 |
| static{} 静态代码块 | 可以有 | 不能有 |
| 本类型之间可扩展 | 单继承 | 多继承 |
| 本类型之间扩展关键字 | extends | extends |

抽象类在被继承时体现的是 is-a 的关系, 接口在被实现时体现的是 can-do 的关系; 与接口相比, 抽象类通常是对同类事物相对具体的抽象, 通常包含抽象方法, 实体方法, 属性变量等; 抽象类是模板式设计, 接口是契约式设计; 在纠结定义接口还是抽象类时, 优先推荐定义为接口, 遵循接口隔离原则, 按某个维度划分为接口, 然后再用抽象类去实现某些接口, 方便后续的扩展和重构

##### 内部类
内部类可分为四种
- 静态内部类: static class StaticInnerClass{};
- 成员内部类: private class InstanceInnerClass{};
- 局部内部类: 定义在方法或者表达式内部
- 匿名内部类: (new Thread(){}).start()

匿名内部类和静态内部类是比较常用的方式; 静态内部类的好处是
- 作用域不会扩散到包外
- 可以通过 "外部类.内部类" 的方式直接访问
- 内部类可以访问外部类中的所有静态属性和方法

##### 访问控制权限
| 访问权限控制符 | 任何地方 | 包外子类 | 包内 | 类内 |
| :---| :--- |:--- |:--- |:--- |
| public | YES | YES | YES | YES |
| protected | NO | YES | YES | YES |
| 无 | NO | NO | YES | YES |
| private | NO | NO | NO | YES |

在定义类时, 推荐访问控制级别从严处理
- 如果不允许外部直接通过 new 创建对象, 构造方法必须是 private
- 工具类不允许有 public 或 default 构造方法
- 类非 static 成员变量与子类共享, 必须是 protected
- 类非 static 成员变量且仅在本类使用, 必须是 private
- 类 static 成员变量且仅在本类使用, 必须是 private
- 如是 static 成员变量, 必须考虑是否为 final
- 类成员方法只供类内部调用, 必须是 private
- 类成员方法只与子类共享, 必须是 protected

##### this 与 super
this 和 super 在很多情况下都是默认省略的
- 本类方法调用本类属性
- 本类方法调用另一个本类方法
- 子类构造器方法隐含调用 super()

this 与 super 的异同点
| - | 不同点 | 相同点 |
| :--- | :--- | :--- |
| this | 访问本类实例属性和方法; 先找本类, 没有则找父类; 单独使用时表示当前对象| 都是关键字, 起指代作用; 在构造方法中必须出现在第一行 |
| super | 由子类访问父类中的实例属性和方法; 直接查找父类; 在子类覆写父类方法时, 访问父类同名方法 | 都是关键字, 起指代作用; 在构造方法中必须出现在第一行 |

##### 类关系
| 类关系 | 英文名 | 描述 | 权力强侧 | 类图 | 示例说明 |
| :--- | :--- | :--- | :--- | :--- |:--- |
| 继承 | Generalization | 父类与子类之间的关系: is-a | 父类方 | 空心三角形 + 实线 | 小狗继承于动物, 完全符合里氏替换原则 |
| 实现 | Realization | 接口与实现类之间的关系: can-do | 接口方 | 空心三角形 + 虚线 | 小狗实现了狗叫的接口行为 |
| 组合 | Generalization | 比聚合更强的关系: contains-a | 整体方 | 实心菱形 + 实线 | 头只能是身体强组合的一部分, 两者完全不可分, 具有相同的生命周期 |
| 聚合 | Generalization | 暂时组装关系: has-a | 组装方 | 空心菱形 + 实线 | 小狗和狗绳之间是暂时聚合关系, 狗绳完全可以复用在另外一条小狗上 |
| 依赖 | Generalization | 一个类用到另一个类: use-a | 被依赖方 | 箭头 + 虚线 | 人喂养小狗, 小狗作为参数传入, 是一种依赖关系 |

##### 序列化
内存中的数据对象只有转换为二进制流才可以进行数据持久化和网络传输, 将数据对象转换为二进制流的过程称为对象的序列化 (Serialization), 反之称为反序列化

#### 方法

##### 方法签名
方法签名包括方法名称和参数列表, 是 JVM 标识方法的唯一索引, 不包括返回值, 访问权限控制符, 异常类型等

##### 参数
Java 中的参数传递都是值复制的传递过程

##### 构造方法
构造方法是方法名与类名相同的特殊方法, 在新建对象时调用, 有如下特征
- 构造方法名称必须与类名相同
- 构造方法没有返回类型
它返回对象的地址, 并赋值给引用变量
- 构造方法不能被继承, 不能被覆写, 不能被直接调用
调用有三种途径: 通过 new 关键字; 在子类的构造方法中通过 super 调用父类的构造方法; 通过反射方式获取
- 类定义时提供了默认的无参的构造方法
但是如果显式定义了有参构造方法, 则此无参构造方法就会被覆盖, 如果依然想拥有就要进行显示定义
- 构造方法可以私有

接口中不能定义构造方法, 在抽象类中可以定义; 在枚举类中, 构造方法是特殊的存在, 不能是 public 修饰, 默认是 private 的, 是绝对的单例, 不允许外部以创建的方式生成枚举对象

##### 类内方法
除了构造方法外, 类中还可以有三类方法: 实例方法, 静态方法, 静态代码块

###### 实例方法
即非静态方法, 依附于某个实际的对象; 当 .class 字节码文件加载之后, 实例方法并不会分配方法入口地址, 只有在对象创建之后才会被分配

###### 静态方法
当类加载后, 即分配了相应的内存空间; 由于生命周期的限制, 使用时需要注意以下两点
- 静态方法中不能使用实例成员变量和实例方法
- 静态方法不能使用 super 和 this 关键字, 因为这两个关键字都是指代需要别创建出来的对象

###### 静态代码块
静态代码块是先于构造方法执行的特殊代码块

##### getter 和 setter
getter 和 setter 方法的好处
- 满足面向对象语言封装的特性
- 有利于统一控制

容易出错的定义方式
- getter/setter 中添加业务逻辑
- 同时定义 isXXX() 和 getXXX()
- 相同的属性名容易带来歧义

##### 同步与异步
同步是阻塞式操作, 必须等待调用方法体执行结束; 异步是非阻塞式操作, 在执行过程中如调用其他方法, 可以不必等待调用完毕而直接结束

##### 覆写
想要成功的覆写父类的方法, 需要满足以下条件: 一大两小两相同
- 访问权限不能变小
- 访问类型能够向上转型成为父类的返回类型
- 异常也要能向上转型成为父类的异常
- 方法名, 参数类型, 参数个数必须严格一致

##### 重载
在同一个类中, 如果多个方法有相同的名字, 不同的参数, 即称为重载
```
public class OverloadMethods {

    public void OverloadMethod() {
        System.out.println("无参方法.");
    }

    public void methodForOverload(int param) {
        System.out.println("参数为基本类型 int 的方法.");
    }

    public void methodForOverload(Integer param) {
        System.out.println("参数为包装类型 Integer 的方法.");
    }

    public void methodForOverload(Integer... param) {
        System.out.println("可变参数方法.");
    }

    public void methodForOverload(Object param) {
        System.out.println("参数为 Object 的方法.");
    }
}
```
如果调用 `methodForOverload(7)` 则会执行哪个方法? JVM 在重载方法中, 选择合适的目标方法的顺序如下
- 精确匹配
- 如果是基本数据类型, 自动转换成更大表示范围的基本类型
- 通过自动装箱和拆箱
- 通过子类向上转型继承路线依次匹配
- 通过可变参数匹配

#### 泛型
泛型的本质是类型参数化, 解决不确定具体对象类型的问题; 在定义时需要注意以下几点
- 尖括号里的每个元素都指代一种未知类型
- 尖括号的位置必须在类名之后或方法返回值之前
- 泛型在定义处只具备执行 Object 方法的能力
- 泛型知识一种编译期的检查, 运行期并没有泛型标记

#### 数据类型

##### 基本数据类型
基本数据类型是指不可再分的原子数据类型, 内存中直接存储这种类型的值, 通过内存地址即可直接访问到数据, 并且此内存区域只能存放这种类型的值; Java 的 9 中基本类型包括 boolean, byte, char, short, int, long, float, double, refvar; refvar 是面向对象世界中的引用变量, 也叫引用句柄

| 序号 | 类型名称 | 默认值 | 大小 | 最小值 | 最大值 | 包装类 | 缓存区间 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | boolean | false | 1B | 0(false) | 1(true) | Boolean | 无 |
| 2 | byte | (byte)0 | 1B | -128 | 127 | Byte | -128 ~ 127 |
| 3 | char | '\u0000' | 2B | '\u0000' | '\uFFFF' | Character | 0 ~ 127 |
| 4 | short | (short)0 | 2B | $ -2^{15} $ | $ 2^{15} $ - 1 | Short | -128 ~ 127 |
| 5 | int | 0 | 4B | $ -2^{31} $ | $ 2^{31} $ - 1 | Integer | -128 ~ 127 |
| 6 | long | 0L | 8B | $ -2^{63} $ | $ 2^{63} $ - 1 | Long | -128 ~ 127 |
| 7 | float | 0.0F | 4B | 1.4e-45 | 3.4e+38 | Float | 无 |
| 8 | double | 0.0D | 8B | 4.9e-324 | 1.798e+308 | Double | 无 |

引用分成两种数据类型: 引用变量本身(refvar) 和引用指向的对象(refobj); refvar 是基本的数据类型, 默认值是 null, 存储的是 refobj 的首地址; refvar 只占 4B 空间, refobj 最少占 16B 空间, 其中对象头占 12B; refobj 实际分为以下三部分存储: 对象头, 实例数据, 对齐填充
```
<------------------- 8 个字节 -----------------><------- 4 个字节 ------->
----------------------------------------------------------------------------------------------------
|哈希码, GC 标记, GC 次数, 同步锁标记, 偏向锁标记 | 类元信息的首地址 (refvar)| 实例数据 ... || 对齐填充 ||
----------------------------------------------------------------------------------------------------
```

##### 包装数据
Boolean 使用静态 final 变量定义; 除了 Float 和 Double 外其他包装类型都有缓存区间; Integer 的缓存区间可以通过 -XX:AutoBoxCacheMax 参数修改; 在使用包装类或者基本数据类的选择上推荐如下
- 所有 POJO 类属性必须使用包装数据类型
- RPC 方法的返回值和参数必须使用包装数据类型
- 所有的局部变量推荐使用基本数据类型

##### 字符串
字符串相关类型主要有: String, StringBuilder, StringBuffer
