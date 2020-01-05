### question

#### 值传递和引用传递
TODO

#### Java 8 的新特性
- Lambda 表达式: 允许把函数作为一个方法的参数
- 方法引用:
- 默认方法: 接口中有一个实现的方法
- Stream API: java.util.stream 真正把函数式编程风格引入到 Java 中
- Date Time API: 加强对日期与时间的处理
- Optional: 用于解决空指针的类
- 新工具: 新的编译工具
- Nashorn, Javascript 引擎

#### 重写 equals 方法为何还要重写 hashCode 方法
在每个覆盖了 equals 方法的类中, 也必须覆盖 hashCode 方法; 如果不这样做的话, 就会违反 Object.hashCode 的通用约定, 从而导致该类无法结合所有基于散列的集合一起正常运作, 这样的集合包括 HashMap, HashSet, HashTable; 以下是 Object 规范
- 在应用程序的执行期间, 只要对象的 equals 方法的比较操作所用到的信息没有被修改, 那么对这同一个对象调用多次, hashCode 方法都必须始终如一的返回同一个整数; 在同一个应用程序的多次执行过程中, 每次执行所返回的整数可以不一致
- 如果两个对象根据 equals 方法比较是相等的, 那么调用这两个对象中任意一个对象的 hashCode 方法都必须产生同样的整数结果
- 如果两个对象根据 equals 方法比较是不相等的, 那么调用这两个对象中任意一个对象的 hashCode 方法, 则不一定要产生不同的整数结果; 但给不相等的对象产生截然不同的整数结果, 有可能提高散列表 (hash table) 的性能

没有覆盖 hashCode 而违反的关键约定是第二条: 相等的对象必须具有相等的散列码

#### Java 中的 fanal 关键字
当 final 修饰一个类时, 表明这个类不能被继承; final 类中的成员变量可以根据需要设为 final, 但是要注意的是 final 类中所有的成员方法都会被隐式地指定为 final 方法  
当 final 修饰一个方法时, 表明以防任何继承类修改它的含义  
当 final 修饰一个变量时, 表明变量在初始化之后不可再修改, 如果是基本类型则其值不能再被修改, 如果是引用类型则其不能再指向其他对象

#### synchronized 和 Lock
synchronized 是 Java 的关键字, 用来修饰以一个方法或者一个代码块, 以保证同一时刻最多只有一个线程执行该段代码; JDK1.5 以后引入了自旋锁, 锁粗化, 轻量级锁, 偏向锁来优化 synchronized 的性能; Lock 是 Java 的一个接口  
synchronized 是内置的语言实现, synchronized 在发生异常时, 会自动释放线程占有的锁, 因此不会导致死锁现象发生; Lock 在发生异常时, 如果没有主动通过 unLock() 来释放锁, 则很可能造成死锁现象, 因此使用 Lock 时需要在 finally 块中释放锁; Lock 可以让等待锁的线程响应中断, 而使用 synchronized 时, 等待的线程会一直等待下去, 不能够响应中断; 通过 Lock 可以知道有没有成功获取锁, 而 synchronized 则无法做到

#### volatile 关键字
TODO

#### 使用 synchronized 关键字修饰一个静态方法和修饰成员方法, 锁的对象有什么不同
synchronized 修饰静态方法, 线程想要执行对应同步代码, 需要获得类锁; synchronized 修饰成员方法, 线程获得是当前调用方法的对象实例的对象锁

#### 面向对象的六原则一法则
TODO
- 单一职责原则
- 开闭原则
- 依赖倒转原则
- 里氏替换原则
- 接口隔离原则
- 合成聚合复用原则
- 迪米特法则

#### static nested class 和 inner class 的不同
TODO

#### 抽象类和接口的区别
- 抽象类 (abstract class)
使用 abstract 修饰的类, 表示一个类没有包含足够多的信息来描述一个具体的对象, 因此也不能够实例化; 但是仍然属于类, 可以定义成员变量, 成员方法, 构造方法等; 一个类中含有抽象方法, 那么这个类必须被声明为抽象类, 抽象方法由子类实现
- 接口 (interface)
接口是抽象方法的一个集合, 一个类通过继承接口的方式, 从而继承并实现接口的抽象方法; 并且不能有成员变量和构造函数

| 区别点 | 抽象类 | 接口 |
| :--- | :--- | :--- |
| 语法 | 可以有成员变量和构造方法 | 不能有成员变量和构造方法 |
| 抽象 | 对类的抽象 | 对行为的抽象 |
| 跨域 | is-a 的继承关系 | has-a 的契约关系 |
| 设计 | 自下而上设计, 有了子类才能抽象出父类 | 自上而下的设计 |

#### final, finally, finalize 的区别
- final 用于声明属性, 方法, 类; 分别表示属性不可变, 方法不可覆盖, 类不可继承
- finally 是异常处理语句结构的一部分, 表示总是执行
- finalize 是 Object 类的一个方法, 在垃圾收集器执行的时候会调用被回收对象的此方法, 可以覆盖此方法提供垃圾收集时的其他资源回收

#### 面向对象的特征有哪些
- 抽象
- 继承
- 封装
- 多态

#### extends 和 super 泛型限定符
泛型上界定义 `<? extends Fruit>`, 泛型下界定义 `<? super Apple>`; 泛型上界的容器只能消费不能生产 (例如, List 只能 get, 不能 add 除 null 以外的对象, 因为不知道是哪个具体子类), 泛型下界的容器只能生产不能消费 (例如, List 只能 add, 不能 get 赋值 Object 以外的类型, 因为不知道是哪个具体的父类); 见以下示例
```java
class Fruit {}
class Apple extends Fruit {}
class Jonathan extends Apple {}

public class GenericCovariantList {

    public static void main(String[] args) {
        // 上界
        List<? extends Fruit> upList = new ArrayList<>();
        Fruit fruit = upList.get(0);
        upList.add(null);
        upList.add(new Fruit()); // 会报错

        // 下界
        List<? super Apple> downList = new ArrayList<>();
        downList.add(new Jonathan());
        Object object = downList.get(0);
        Apple apple = downList.get(0); // 会报错
    }
}
```
归根结底是因为支持向上转型而不支持向下转型

#### 什么是泛型
