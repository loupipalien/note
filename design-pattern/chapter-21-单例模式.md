### 单例模式
单例模式即保证一个类仅有一个实例, 并提供一个访问它的全局访问点  
通常可以让一个全局变量使得一个对象被访问, 但它不能防止实例化多个对象; 一个最好的办法就是, 让类自身负责保存它的唯一实例, 并且提供一个访问该实例的方法

#### 懒汉式 (线程不安全)
```
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
以上代码看似满足了单例模式的要求, 但是有一个致命的问题是线程不安全; 在多线程并发调用 getInstance() 方法时可能会创建多个实例

#### 懒汉式 (线程安全)
```
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        // 或代码块加锁
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
既然在多线程并发调用下有问题, 那么第一反应应该是加上锁保证线程安全, 给方法加锁, 给代码块加锁都是可以的; 但这样实现的问题是效率不高, 因为每次获取实例时都要先获得锁

#### 懒汉式-双重检查锁 (线程不安全)
```
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        // Double Checked Locking (DCL)
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
这里使用双重检查来避免每次获取时都要获取锁的低效步骤, 但是以上实例是不安全的; 以上代码中的 `instance = new Singleton();` 语句在 JVM 中执行时被拆解为三步
```
memory = allocate();  // 1: 分配对象的内存空间
ctroInstance(memory); // 2: 初始化对象
instance = memory;    // 3: 设置 instance 指向刚分配的内存地址
```
Java 程序中所有线程执行代码时都遵守 instr-thread semantics, 即允许那些不会改变单线程程序执行结果的重排序; 以上指令在 JVM 中执行时可能会重排序
```
memory = allocate();  // 1: 分配对象的内存空间
instance = memory;    // 3: 设置 instance 指向刚分配的内存地址 (执行完这一步就为 非 null 了, 但此时实例尚未初始化)
ctroInstance(memory); // 2: 初始化对象
```
单线程执行这样的重排序不会影响执行; 当多线程时, 线程 A 在 3 指令运行后, 2 指令运行前, 线程 B 可判断不为 null 而获取到实例, 但由于尚未初始化, 在使用时可能导致错误

#### 懒汉式 - 基于 volatile 内存语义避免重排序 (线程安全)
```
public class Singleton {
    // 使用 volatile 修饰
    private volatile static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        // Double Checked Locking (DCL)
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
指定 volatile 后, 会在某些指令操作间插入内存屏障, 指令重排会被禁止; 在多线程环境下便是安全的了 (使用 JDK 5 之前的版本会有问题, 而 JDK 5 后, JSR-133 增强了 volatile 语义, 可保证线程安全)

#### 懒汉式 - 基于类初始化加锁 (线程安全)
```
public class Singleton {
    private static class Holder {
        public static Singleton instance = new Singleton();
    }


    private Singleton() {}

    public static Singleton getInstance() {
        return Holder.instance;
    }
}
```
JVM 在类的初始化阶段 (即在 Class 被加载之后, 且被线程使用之前), 会执行类的初始化; 在执行类的初始化期间, JVM 会去获取一个锁, 这个锁可以同步多个线程对同一个类的初始化; 而初始化一个类, 包括执行这个类的静态初始化和初始化在这个类中声明的静态字段; 根据 Java 语言规范, 在首次发生下列任意一种情况时, 一个类或接口类型 T 将立即被初始化
- T 是一个类, 而且一个 T 类型的实例被创建
- T 是一个类, 且 T 中声明的一个静态方法被创建
- T 中声明的一个静态字段被赋值
- T 中声明的一个静态字段被使用, 而且这个字段不是一个常量字段
- T 是一个顶级类, 而且一个断言语句嵌套在 T 内存被执行

为了实现懒加载, 将实例放在一个静态内部类中, 当调用 getInstance() 时才触发 Holder 类的初始化, 达到延迟初始化的效果

#### 饿汉式 - 基于类初始化加锁 (线程安全)
```
public class Singleton {
    private static Singleton instance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}
```
当然也可以不使用静态内部类, 使用 Singleton 类初始化保证线程安全; 这样饿汉式的实例初始化方式, 在依赖外部参数或配置文件时是无法使用的, 因为尚未获取参数或尚未加载文件就已经开始初始化了

#### 枚举
枚举创建默认是线程安全的, 且枚举创建十分简单; 但是并不常用, 因为枚举并不是为单例模式设计的...
```
public enum Singleton {
    INSTANCE;
}
```

#### 小结
推荐使用 `懒汉式 - 基于 volatile 内存语义避免重排序 (线程安全)` 和 `懒汉式 - 基于类初始化加锁 (线程安全)`

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)  
[Java 并发编程的艺术](https://book.douban.com/subject/26591326/)  
[如何正确地写出单例模式](http://wuchong.me/blog/2014/08/28/how-to-correctly-write-singleton-pattern/)
