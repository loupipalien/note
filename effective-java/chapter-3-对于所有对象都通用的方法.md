### 对于所有对象都通用的方法
尽管 Object 是一个具体类, 但是设计它主要是为了扩展; 它所有的非 final 方法 (equals, hahsCode, toString, clone, finalize) 都有明确的通用约定 (general contract), 因为它们被设计为要被覆盖的; 任何一个类在覆盖这些方法的时候, 都要遵守这些通用约定, 如果不能做到这一点, 其他依赖于这些约定的类 (如 HashMap 和 HashSet) 就无法结合该类一起正常工作

#### 第 8 条: 覆盖 equals 时请遵守通用约定
覆盖 equals 方法看起来简单但是许多覆盖方式会导致错误; 在不覆盖 equals 方法的情况下, 类的每个实例都只与自身相等; 如果满足了以下任何一个条件, 这就是期望的结果
- 类的每个实例本质上都是唯一的
对于代表活动实体而不是值的类来说却是如此, 例如 Thread; Object 提供的 equals 实现对于这些类来说正是正确的行为
- 不关心类是否提供了 "逻辑相等" 的测试功能
java.util.Random 覆盖了 equals, 以检查两个 Random 实例是否产生相同的随机数序列, 但是设计者并不认为客户需要或者期望这样的功能; 在这样的情况下, Object 提供的 equals 实现已经足够了
- 超类已经覆盖了 equals, 从超类继承过来的行为对于子类也是合适的
如大多数 Set 实现都从 AbstractSet 继承 equals 实现, List 实现从 AbstractList 继承 equals 实现, Map 实现从 AbstractMap 继承 equals 实现
- 类是私有的或是包级私有的, 可以确定它的 equals 方法永远不会被调用
在这种情况下无疑是应该覆盖 equals 方法, 以防它被意外调用

如果类具有自己特有的 "逻辑相等" 的概念 (不等同与对象等同的概念), 而且超类还没有覆盖 equals 以实现期望的行为, 这时我们就需要覆盖 equals 方法; 这通常属于 "值类 (value class)" 的情形, 值类仅仅是一个表示值的类, 例如 Integer 或者 Date, 程序员在利用 equals 方法来比较值对象的引用时, 希望知道它们在逻辑上是否相等, 而不是想了解它们是否指向同一个对象; 为了满足这个需求, 不仅必需覆盖 equals 方法, 而且这样做也使得这个类的实例可以被用做映射表 (map) 的键 (key), 或者集合 (set) 的元素, 使得映射或者集合表现出预期的行为  
有一种 "值类" 不需要覆盖 equals 方法, 即用实例受控 (见第 1 条) 确保 "每个值至多存在一个对象" 的类, 枚举类型 (见第 30 条) 就属于这样的类; 对于这样的类而言, 逻辑相同与对象等同是一回事, 因此 Object 的 equals 方法等同于逻辑意义上的 equals 方法  
在覆盖 equals 方法时必须要遵守以下规定, equals 方法实现了等价关系 (equivalence relation)
- 自反性 (reflexive): 对于任何非 null 的引用值 x, x.equals(x) 必须返回 true
- 对称性 (symmetric): 对于任何非 null 的引用值 x 和 y, 当且仅当 y.equals(x) 返回 true 时, x.equals(y) 必须返回 true
- 传递性 (transitive): 对于任何非 null 的引用值 x, y, z, 如果 x.equals(y) 放回 true, 并且 y.equals(z) 也返回 true, 那么 x.equals(z) 也必须返回 true
- 一致性 (consistent): 对于任何非 null 的引用值 x 和 y, 只要 equals 的比较操作在对象中所用的信息没有被修改, 多次调用 x.equals(y) 就会一致地返回 true, 或者一致的返回 false
- 对于任何非 null 的引用值 x, x.equals(null) 必须返回 false

结合以上所有要求, 得出以下实现高质量 equals 方法的诀窍
- 使用 == 操作符检查 "参数是否为这个对象的引用"
- 使用 instanceof 操作符检查 "参数是否为正确的类型"
- 把参数转换成正确的类型
- 对于该类中的每个 "关键 (significant)" 域, 检查参数中的域是否与该对象中对应的域相匹配
- 当编写完成 equals 方法之后, 应该要问自己三个问题: 它是否是对称, 传递的, 一致的?
  - 覆盖 equals 时总要覆盖 hashCode (见第 9 条)
  - 不要企图让 equals 方法过于智能
  - 不要将 equals 声明中的 Objects 对象替换为其他的类型

#### 第 9 条: 覆盖 equals 时总要覆盖 hashCode
一个很常见的错误根源在于没有覆盖 hashCode 方法, 在每个覆盖了 equals 方法的类中, 也必须覆盖 hashCode 方法; 如果不这样做的话, 就会违反 Object.hashCode 的通用约定, 从而导致该类无法结合所有基于散列的集合一起正常运作, 包括 HashMap, HashSet, HashTable; 以下是摘自 Object 规范的约定内容
- 在应用侧滑盖女婿的执行期间, 只要对象的 equals 方法的比较操作所用到的信息没有被修改, 那么对这同一个对象调用多次, hashCode 方法都必须时钟如一地返回同一个整数; 在同一个应用程序的多次执行过程中, 每次执行所返回的整数可以是不同的
- 如果两个对象根据 equals(Object) 方法的比较是相等的, 那么调用这两个对象中任意一个对象的 hashCode 方法都必须产生相同的整数结果
- 如果两个对象根据 equals(Object) 方法的比较是不相等的, 那么调用这两个对象中任意一个对象的 hashCode 方法, 则不一定要产生不同的整数结果; 但是程序员应该知道, 给不相等的对象产生截然不同的整数结果, 有可能提高散列表的性能
