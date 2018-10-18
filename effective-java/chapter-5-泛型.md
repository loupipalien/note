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
从技术的角度来说, E, List<E>, List<String> 这样的类型应称作不可以具体化的类型, 不可具体化类型是指其运行时表示fa所包含的信息比它编译时表示法包含的信息更少的类型; 唯一可具体化的参数类型是无限制的通配符类型 (如 List<?>, Map<?,?>), 虽不常用, 但是创建无限制通配类型的数组是合法的
