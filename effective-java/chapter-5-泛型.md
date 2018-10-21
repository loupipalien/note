### 泛型

#### 第 23 条: 请不要在新代码中使用原生态类型
声明具有一个或者多个类型参数的类或者接口, 就是泛型 (generic) 类或者接口; 而每个泛型都有一个原生态类型, 即不带任何实际参数的泛型名称, 例如 List 是 List<String> 的原生态类型; 如果不提供类型参数, 使用泛型也仍然是合法的, 但是不应该这样做, 因为使用原生态类型失去了泛型在安全性和表述性方面的所有优势; 在使用中可以将 List<String> 引用传递给类型 List, 但是不能传递给 List<Object>, 因为泛型有子类型化的规则, List<String> 是原生态类型 List 的一个子类型, 而不是 List<String> 的子类型  
由于使用原生类型很危险, Java 提供了一种安全的替代方法, 称作无限制通配符类型 (unbouned wildcard type), 如果要使用泛型, 但不确定或者不关心实际的类型参数, 就可以使用一个问号代替; 无限制通配符类型与原生态类型的区别是: 无限制通配符类型类型是安全的, 原生态类型不安全, 可以将任何元素放入原生态类型的集合中, 但是不能将任何元素 (除了 null) 放入无限制通配符类型的集合中  
不要在新代码中使用原生态类型, 但有两个源于 "泛型信息可以在运行是被擦除" 的例外
- 在类文字中必须使用原生态类型, 规范不允许使用参数化类型 (虽然允许数组类型和基本类型), List.class 是合法的, 但 List<String>.class 是非法的
- 在 instanceof 操作符时使用原生态类型, 参数化类型而非无限制通配符类型上使用 instanceof 操作符是非法的

一些术语介绍
| 术语 |示例|所在条目|
|---|---|---|
|参数化类型|List<String>|第 23 条|
|实际类型参数|String|第 23 条|
|泛型|List<E>|第 23, 26 条|
|形式类型参数|E|第 23 条|
|无限制通配符类型|List<?>|第 23 条|
|原生态类型|List|第 23 条|
|有限制类型参数|List<E extends Number>|第 26 条|
|递归类型限制|List<T extends Comparable<T>>|第 27 条|
|有限制通配符类型|List<? extends Number>|第 28 条|
|泛型方法|static <E> List<E> asList(E[] a)|第 27 条|
|类型令牌|String.class|第 29 条|

#### 第 24 条: 消除非受检警告
用泛型编程时, 会遇到许多编译器的警告: 非受检强制转化警告 (unchecked cast warings), 非受检方法调用警告, 非受检普通数组创建警告, 非受检转换警告 (unchecked conversion warnings); 要尽可能到消除这些警告, 以保证代码类型安全, 确保不会在运行时出现 ClassCastException 异常, 如果无法消除警告, 同时可以证明引起警告的代码是类型安全的, 这种情况下才可以用一个 @SupressWarings("unchecked") 注解禁止这条警告  
@SupressWarings 注解可以用在任何粒度的级别中, 从单独的局部变量声明到整个类都可以, 但应该始终尽可能小的范围中使用 @SupressWarings 注解, 通常是在变量上或者方法上, 尽量不要在整个类上使用, 这样可能掩盖了重要的警告; 每当使用 @SupressWarings("unchecked") 注解时, 都要添加一条注解, 说明这样做为什么是安全的, 帮助阅读代码

#### 第 25 条: 列表优先于数组
数组与泛型相比有两个重要的不同点; 首先数组是协变的 (covariant): 表示如果 Sub 为 Super 的子类型, 那么数组类型 Sub[] 是 Super[] 的子类型; 相反, 泛型是不可变的 (invariant): 对于任意两个不同类型 Type1 和 Type2, List<Type1> 即不是 List<Type2> 的子类型, 也不是 List<Type2> 的超类型; 实际上数组是有缺陷的, 以下两种都不能将 String 放入 Long 容器中, 但是数组在运行时才报错, 而泛型在编译时即报错
```
// Fails at runtime
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in" // Throws ArrayStoreException

// Won't compile
List<Object> ol = new Arraylist<Long>(); // Incompatible types
ol.add("I don't fit in");
```
数组与泛型之间的第二大区别在于, 数组是具体化的, 因此数组会在运行时才知道并检查它们的元素类型约束; 如上所述, 企图将 String 保存到 Long 数组中会得到一个 ArrrayStoreException 异常; 相比之下, 泛型是通过擦除来实现的, 在编译时强化它们的类型信息, 并在运行时丢弃它们的类型信息  
由于以上区别, 数组和泛型不能很好的混用, 如创建泛型, 参数化类型或者类型参数的数组是非法的, new List<E>[], new List<String>[], new E[] 都是非法的, 编译时都会导致一个 generic array creation (泛型数组创建) 错误; 创建泛型数组是非法的, 因为它不是类型安全的; 如果它合法, 那么编译器在其他正确的程序中发生的转换就会在运行时失败, 并出现一个 ClassCastException 异常, 这就违背了泛型系统提供的基本保证; 见以下示例代码片段
```
// Why generic array creation is illegal - won't compile

// 如果创建泛型数组是合法的
List<String>[] stringlists = new List<String>[1];
// 创建并初始化了一个包含单个元素的 List<Integer>
List<Integer> intList = Arrays.asList(42);
// 将 List<String> 数组保存到 Object 数组中, 因为泛型擦除, 并且数组是协变的, 所以是合法的
Object[] objects = stringLists;
// 将 List<Integer> 保存到 Object 数组中, 因为泛型擦除, List<Integer> 实例保存到 List<String>[] 不会抛出 ArrayStoreException 异常
objects[0] = intList;
// 抛出 ClassCastException 异常
String s = stringLists[0].get(0);
```
所以为了防止这种情况出现, (创建泛型数组) 第一行产生了一个编译时错误  
从技术的角度来说, E, List<E>, List<String> 这样的类型应称作不可以具体化的类型, 不可具体化类型是指其运行时表示fa所包含的信息比它编译时表示法包含的信息更少的类型; 唯一可具体化的参数类型是无限制的通配符类型 (如 List<?>, Map<?,?>), 虽不常用, 但是创建无限制通配类型的数组是合法的; 当泛型数组创建错误时最好的解决办法通常是优先使用集合类型 List<E>, 而不是数组类型 E[], 这样可能会损失一些性能或者简洁性, 但是换回的是更高的类型安全和互用性  
假设有一个 (Collections.synchronizedList 返回的那种) 同步列表和一个函数 (它有两个与该列的元素同类型的参数值, 并返回第三个值), 现在假设要编写一个方法 reduce, 并使用函数 apply 来处理这个列表
```
// Reduction without generics, and with concurrency flaw!
static Object reduce(List list, Function f, Object initVal) {
    synchronized(list) {
        Object reduce = initVal;
        for (Object o : list) {
            result = f.apply(result, o);
        }
        return result;
    }
}

interface Function {
    Object apply(Object arg1, Object arg2);
}
```
假设现在已经读过第 67 条, 它告诉你不要从同步区域中调用 "外来的 (alien)" 方法, 在持有锁的时候修改 reduce 方法来复制表中的内容, 也可以让你在备份上执行减法; 在 JDK5 发行版本之前, 要这么做一般是利用 List 的 toArray 方法 (它在内部锁定列表)
```
// Reduction without generics or concurrency flaw
static Object reduce(List list, Function f, Object initVal) {
    synchronized(list) {
        Object[] snapshot = list.toArray() // Locks list internally
        Object reduce = initVal;
        for (Object o : snapshot) {
            result = f.apply(result, o);
        }
        return result;
    }
}
```
如果试图通过泛型来完成这一点, 就会遇到一些警告的麻烦; 以下是 Function 接口的泛型版
```
interface Function<T> {
    T apply(T arg1, T arg2);
}

// Naive generics version of reduction -

static Object reduce(List<E> list, Function<E> f, E initVal) {
    /**
    * E[] snapshot = list.toArray() // Naive generics version of reduction - won't  compile
    * E[] snapshot = (E[]) list.toArray() // Naive generics version of reduction - unchecked cast
    */
    List<E> snapshot;
    synchronized (list) {
        snapshot = new Arraylist<E>(list);
    }
    E reduce = initVal;
    for (E o : snapshot) {
        result = f.apply(result, o);
    }
    return result;
}
```
使用列表转换虽然代码更冗长一些, 但是可以确定在运行时不会得到 ClassCastException 异常  
总而言之, 数组和泛型有着非常不同的类型规则, 数组是协变且具体化的, 泛型是不可变的且可以擦除的; 因此数组提供了运行时的类型安全, 但是没有编译时的类型安全; 一般来说, 数组和泛型不能很好的混用, 如果发现混起来用得到了编译时错误或者警告, 第一反应应该是用列表代替数组

#### 第 26 条: 优先考虑泛型
简单的堆栈实现
```
// Object-based collection - a prime cnadidate for generics
public class stack  {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
       elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

    public boolean siEmpty() {
        retrun size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```
泛型实现
```
// Initial attempt to generify Stack - won't compile
public class stack<E>  {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        // generic array creation
        elements =  new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    public boolean siEmpty() {
        retrun size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```
以上代码在泛型数组创建时会报错, 因为不可创建不可具体化类型的数组; 解决这个问题通常有两种办法, 第一种是直接绕过创建泛型数组的禁令, 创建一个 Object 数组然后在转换为泛型数组
```
/**
 * The elements array will contain only E instances from push(E),
 * this is sufficient to ensure type safety, but the runtime type of
 * the array won't be E[], it will always be Object[]!
 */
@SuppressWaring("unchecked")
public Stack() {
    elements =  (E[]) new E[DEFAULT_INITIAL_CAPACITY];
}
```
消除 Stack 中泛型数组创建错误的第二种方法是, 将 elements 域的类型从 E[] 改为 Object[]; 这样需要将类型转换的警告放在变量上, 因为 E 是一个不可具体化的类, 编译器无法在运行时检验转换
```
public E pop() {
    if (size == 0) {
        throw new EmptyStackException();
    }
    /*
     * push required elements to be of type E, so cast is correct
     */
    @SuppressWaring("unchecked")
    E result = (E) elements[--size];
    elements[size] = null;
    return result;
}
```
具体选择哪种处理方式主要肯个人偏好; 但禁止数组类型的未受检转换比变量类型转换更加危险, 所以建议采用第二种方案; 但是在实际 Stack 代码中需要变量转换的地方太多, 反而第一种方案更常用  
有一些泛型限制了可允许的类型参数值, 例如 java.util.concurrent.Delayed 的一个子类型, 其声明如下
```
class DelayQueue<E extends Delayed> implements BlockingQueue<E>;
```
类型参数列表 (<E extends Delayed>) 要求实际的参数 E 必须是 java.util.concurrent.Delayed 的一个子类型, 它允许 DelayQueue 实现及客户端 DelayQueue 的元素上利用 Delayed 方法, 无需显式转换, 也没有出现 ClassCastException 的风险; 类型参数 E 被称作有限制的类型参数 (bounded type parameter)  
总而言之, 使用泛型比使用需要的在客户端代码中进行转换的类型来得更安全, 也更加容易; 再设计类型的时候, 确保它们不需要转换就可以使用

#### 第 27 条: 优先考虑泛型方法
