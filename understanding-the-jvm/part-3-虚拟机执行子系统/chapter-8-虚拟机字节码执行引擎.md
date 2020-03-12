### 虚拟机字节码执行引擎
代码编译的结果从本地机器码转变为字节码, 是存储格式发展的一小步, 却是编程语言发展的一大步

#### 概述
执行引擎是 Java 虚拟机最核心的组成部分之一, 虚拟机是一个相对于物理机的概念, 这两种机器都有代码执行能力, 其区别是物理机的执行引擎是直接建立在处理器, 硬件, 指令集和操作系统层面上的, 而虚拟机的执行引擎则是自己实现的, 因此可以自行制定指令集与执行引擎的结构体系, 并且能够执行那些不被硬件直接支持的指令集格式  
在 Java 虚拟机规范中制定的了字节码执行引擎的概念模型, 在不同的虚拟机实现里, 执行引擎在执行 Java 代码的时候可能会有解释执行 (解释器执行) 和编译执行 (通过即使编译器产生本地代码执行) 两种选择; 但所有的 Java 虚拟机的执行引擎都是一致的: 输入的是字节码文件, 处理过程是字节码解析的等效过程, 输出的是执行结果

#### 执行时栈帧结构
栈帧 (Stack Frame) 是用于支持虚拟机进行方法调用和方法执行的数据结构, 它是虚拟机运行时数据区中的虚拟机栈 (Virtual Machine Stack) 的栈元素; 栈帧存储了方法的局部变量表, 操作数栈, 动态连接和方法返回地址等信息; 每一个方法从调用开始至执行完成的过程, 都对应着一个栈帧在虚拟机栈中从入栈到出栈的过程  
每一个栈帧都包括了局部变量表, 操作数栈, 动态连接, 方法返回地址和一些额外的附加信息; 在编译程序代码时, 栈帧中需要多大的局部变量表, 多深的操作数栈都已经完全确定了, 并且写入到方法表的 Code 属性中, 因此一个栈帧需要分配多少内存, 不会受到程序运行期变量数据的影响, 而仅仅取决于具体的虚拟机实现  
一个线程中的方法调用链可能会很长, 很多方法都同时处于执行状态; 对于执行引擎来说, 在活动线程中, 只有位于栈顶的栈帧才是有效的, 称为当前栈帧, 与这个栈帧相关联的方法称为当前方法; 执行引擎运行的所有字节码指令都只针对当前栈帧进行操作

##### 局部变量表  
局部变量表 (Local Variable Table) 是一组变量值存储空间, 用于存放方法参数和方法内部定义的局部变量; 在 Java 程序编译为 Class 文件时就在方法的 Code 属性的 max_locals 数据项中确定了该方法所需要分配的局部变量表的最大容量  
局部变量表的容量以变量槽 (Variable Slot) 为最小单位, 虚拟机规范中并没有明确指明一个 Slot 应占用的内存空间大小, 只是很有导向性地说到每个 Slot 都应该能存放一个 boolean, byte, char, short, int, float, reference 或 returnAddress 类型的数据; 这 8 种数据类型都可以使用 32 位或更小的物理内存来存放, 但这种描述与明确指出 "每个 Slot 占用 32 位长度的内存空间" 是有一定差别的  
所说的 8 种数据类型前 6 种按照 Java 语言种对应数据类型的概念去理解即可, 第 7 种 reference 类型表示对一个对象实例的引用, 虚拟机规范既没有说明它的长度, 也没有明确指出这种引用应有怎样的结构; 但一般来说, 虚拟机实现至少都应当能通过这个引用做到两点: 一是从此引用中直接或间接的查找到对象在 Java 堆中的数据存放的起始地址索引, 二是此引用中直接或间接的查找到对象所属数据类型在方法区中的存储的类型信息, 否则无法实现 Java 语言规范定义的语法约束; 第 8 种即 returnAddress 类型已经很少见了, 它是位字节指令 jsr, jsr_w 和 ret 服务的, 指向了一条字节码指令的地址, 很古老的 Java 虚拟机曾经使用这几条指令来实现异常处理, 现在已经由异常表代替  
对于 64 位的数据类型, 虚拟机会以高位对齐的方式位其分配两个连续的 Slot 空间, Java 语言中明确的 (reference 类型则可能是 32 位也可能是 64 位) 64 位数据类型只有 long 和 duoble 两种; 这里把 long 和 double 数据类型分割存储的做法与 "long 和 double 的非原子性协定" 中把一次 long 和 double 数据类型读写分割成两次 32 位读写的做法类似; 由于局部变量表建立在线程的堆栈上, 是线程私有的数据, 无论读写两个连续的 Slot 是否位原子操作, 都不会引起数据安全问题  
在方法执行时, 虚拟机是使用局部变量表完成参数值到参数变量列表的传递过程, 如果执行的是实例方法 (非 static 的方法), 那局部变量表中第 0 位索引的 Slot 默认是用于传递方法所属实例的引用, 在方法中可以通过关键字 "this" 来访问到这个隐含的参数, 其余参数则按照参数表顺序排列, 参数表分配完毕后, 再根据方法体内部定义的变量顺序和作用域分配其余 Slot  
为了尽可能节省栈帧空间, 局部变量表中的 Slot 是可以重用的, 方法体中定义的变量, 其作用域并不一定会覆盖整个方法体, 如果当前字节码 PC 计数器的值已经超出了某个变量的作用域, 那这个变量对应的 Slot 就可以交给其他变量使用; 这样的设计 除了节省栈帧空间以外, 还伴随着一些额外的副作用; 例如, 在某些情况下, Slot 复用会直接影响到系统的垃圾收集行为
```
public static void main(String[] args) {
    byte[] placeHolder = new byte[64 * 1024 * 1024];
    System.gc();
}

// GC 信息如下
[GC (System.gc())  68865K->66288K(125952K), 0.0007677 secs]
[Full GC (System.gc())  66288K->66222K(125952K), 0.0178481 secs]
```
执行 System.gc() 时, 变量 placeHolder 还在作用域内, 虚拟机自然是不会回收 placeHolder 的内存的; 继续把代码修改如下
```
public static void main(String[] args) {
    {
        byte[] placeHolder = new byte[64 * 1024 * 1024];
    }
    System.gc();
}

// GC 信息如下
[GC (System.gc())  68865K->66320K(125952K), 0.0011311 secs]
[Full GC (System.gc())  66320K->66222K(125952K), 0.0045486 secs]
```
placeHolder 作用域被限制在花括号内, 从代码逻辑上讲, 在执行 System.gc() 时, placeHolder 已经不能再被访问了, 当内存仍然还没有被回收; 继续修改代码如下
```
public static void main(String[] args) {
    {
        byte[] placeHolder = new byte[64 * 1024 * 1024];
    }
    int a = 0;
    System.gc();
}

// GC 信息如下
[GC (System.gc())  68865K->66320K(125952K), 0.0011931 secs]
[Full GC (System.gc())  66320K->686K(125952K), 0.0045929 secs]
```
这个修改看起来很莫名奇妙, 但是运行发现 GC 时 placeHolder 的内存真正的被回收了  
placeHolder 能被回收的根本原因是: 局部变量表中的 Slot 是否还存有关于 placeHolder 数据对象的引用; 第一次修改中, 代码虽然已经离开了 placeHolder 的作用域, 但在此之后, 没有任何对局部变量表的读写操作, placeHolder 原本所占用的 Slot 还没有被其他变量所复用, 所以作为 GC Root 一部分的局部变量表仍然保持着对它的关联; 这种关联没有即时被打断在大部分情况下影响都很轻微  
关于局部变量表, 还有一点可能会对实际开发产生影响, 就是局部变量不像前面介绍的类变量那样存在 "准备阶段", 类变量有两次赋初始值的过程, 因此在初始化阶段没有为类变量赋值, 类变量仍然具有一个确定的初始值; 但局部变量就不一样, 如果一个局部变量定义了但没有赋初始值是不能使用的

##### 操作数栈
操作数栈 (Operand Stack) 也常称为操作栈, 是一个后入先出 (Last In First Out, LIFO) 栈; 同局部变量表一样, 操作数栈的最大深度也在编译的时候写入到 Code 属性的 max_stacks 数据项中; 操作数栈的每一个元素可以是任意的 Java 数据类型, 包括 long 和 double; 32 位数据类型占栈容量为 1, 64 位数据类型所占栈的容量为 2; 在方法执行的任何时候, 操作数栈的深度都不会超过 max_stacks 数据项中设定的最大值
当一个方法刚刚开始执行的时候, 着方法的操作数栈是空的, 在方法的执行过程中, 会有各种字节码指令往操作数栈中写入和提取内容, 也就是入栈/出栈操作; 操作数栈中元素的数据类型必须与字节码指令的序列严格匹配, 在编译程序代码时, 编译器要严格保证这一点, 在类校验阶段的数据流分析中还要再次验证  
在概念模型中, 两个栈帧作为虚拟机栈的元素, 是完全互相独立的; 但在大多数虚拟机的实现里都会做一些优处理, 令两个栈帧出现一部分重叠; 让下面栈帧的部分操作数栈与上面栈帧的部分局部变量表重叠在一起, 这样在进行方法调用时就可以共用一部分数据, 无须进行额外的参数赋值传递; Java 虚拟机的解释执行引擎称为 "基于栈的执行引擎", 其中所指的 "栈" 就是操作数栈

##### 动态连接
每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用, 持有这个引用是为了支持方法调用过程中的动态连接 (Dynamic Linking); Class 文记的常量池中存有大量的符号引用, 字节码中的方法调用指令就以常量池中指向方法的符号引用作为参数; 这些符号引用一部分会在类加载阶段或第一次使用的时候就转化为直接引用, 这种转化称为静态连接; 另外一部分将在每一次运行期间转化为直接引用, 这部分称为动态连接

##### 方法返回地址
当一个方法开始执行后, 只有两种方式可以退出这个方法
- 执行引擎遇到任意一个方法返回的字节码指令, 这个时候可能会有返回值传递给上层的方法调用者 (调用当前方法的方法称为调用者), 是否有返回值和返回值的类型将根据遇到何种方法返回指令来决定, 这种退出方法的方式称为正常完成出口 (Normal Method Invocation Completion)
- 在方法执行过程中遇到了异常, 并且这个异常没有在方法体内得到处理, 无论是 Java 虚拟机内存产生的异常, 还是代码中使用 athrow 字节码指令产生的异常, 只要在本方法的异常表中没有搜索到匹配的异常处理器, 就会导致方法退出, 这种退出方法的方式称为异常完成出口 (Abrupt Method Invocation Completion); 一个方法使用异常完成出口的方式, 是不会给它的上层调用者产生任何返回值的

##### 附加信息
虚拟机规范允许具体的虚拟机实现增加一些规范中没有描述的信息到栈帧中, 例如调试相关的信息, 这部分信息完全取决于具体的虚拟机实现

#### 方法调用
方法调用并不等同于方法执行, 方法调用阶段唯一的任务就是确定被调用方法的版本 (即调用哪一个方法), 暂时还不涉及方法内部的具体运行过程; Class 文件的编译过程中不包含传统编译中的连接步骤, 一切方法调用在 Class 文件里存储的都只是符号引用, 而不是方法在实际运行时内存布局的入口地址 (相当于直接引用); 这个特性给 Java 带来了更强大的动态扩展能力, 但也使 Java 方法调用过程变得相对复杂起来, 需要在类加载期间, 甚至到运行期间才能确定目标方法的直接引用

##### 解析
所有方法调用中的目标方法在 Class 文件里都是一个常量池中的符号引用, 在类加载的解析阶段, 会将其中的一部分符号引用转化为直接引用, 这种解析能成立的前提是: 方法在程序真正运行之前就有一个可确定的调用版本, 并且这个方法的调用版本在运行期是不可改变的; 换句话说, 调用目标在程序代码写好, 编译器进行编译时就必须确定下来, 这类方法的调用称为解析 (Resolution ) 调用
在 Java 语言中符合 "编译期可知, 运行期不可变" 要求的方法, 主要包括静态方法和私有方法两类, 前者与类型直接关联, 后者在外部不可访问, 这两种方法各自的特点决定了它们不可能通过继承或别的方式重写其他版本, 因此它们都合适在类加载阶段进行解析; 与之对应的是, 在 Java 虚拟机里提供了 5 条方法调用字节码指令, 分别如下
- invokestatic: 调用静态方法
- invokespecial: 调用实例构造器 <init> 方法, 私有方法和父类方法
- invokevirtual: 调用所有的虚方法
- invokeinterface: 调用接口方法, 会在运行时再确定一个实现此接口的对象
- invokedynamic: 先在运行时动态解析出调用点限定符所引用的方法, 然后再执行该方法, 在此之前的 4 条调用指令,　分派逻辑是固化在 Java 虚拟机内部的, 而 invokedynamic 指令的分派逻辑是由用户所设定的引导方法决定的

只要能被 invokestatic 和 invokespecial 指令调用的方法, 都可以在解析阶段中确定唯一的调用版本, 符合这个条件的有静态分方法, 私有方法, 实例构造器, 父类方法 4 类, 它们在类加载的时候就会把符号引用解析为该方法的直接引用; 这些方法可以称为非虚方法, 与之相对的其他方法称为虚方法 (除去 final 方法)  
Java 中的非虚方法除了使用 invokestatic, invokespecial 调用的方法之外, 还有一种就是被 final 修饰的方法; 虽然 final 方法是使用 invokevirtual 指令来调用的, 但是由于它无法被覆盖, 没有其他版本, 所以也无须对方法接收者进行多态选择, 又或者说多态的选择结果肯定是唯一的, 在 Java 语言规范中明确说明了 final 方法是一种非虚方法  
解析调用一定是个静态的过程, 在编译期间就完全确定, 在类装载的解析阶段就会把涉及的符号引用全部转变为可确定的直接引用, 不会延迟到运行期再去完成; 而分派 (Dispatch) 调用则可能是静态的也可能是动态的, 根据分派依据的 [宗量] 数可分为单分派和多分派; 因此这两类分派方式的两两组合就构成了静态单分派, 静态多分派, 动态单分配, 动态多分派 4 中分派组合情况

##### 分派
Java 是一门面向对象的程序语言, 因为 Java 具备面向对象的三个基本特征: 继承, 封装, 多态; 分派调用过程将会解释多态性的一些最基本的体现, 如重写和重载在 Java 虚拟机中是如何实现的, Java 虚拟机如何确定正确的目标方法  

###### 静态分派
```Java
public class StaticDispatch {

    static abstract class Human {}

    static class Man extends Human {}

    static class Woman extends Human {}

    public void sayHello (Human guy) {
        System.out.println("Hello, guy!");
    }

    public void sayHello (Man guy) {
        System.out.println("Hello, gentleman!");
    }

    public void sayHello (Woman guy) {
        System.out.println("Hello, lady!");
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        StaticDispatch sd = new StaticDispatch();
        sd.sayHello(man);
        sd.sayHello(woman);
    }

}
```
在 `Human man = new Man()` 的代码中, 我们把 Human 的变量称为静态类型 (Static Type), 或者叫做外观类型 (Apprent Type), 后面的 Man 则称为变量的实际类型 (Actual Type); 静态类和实际类型在程序中都会发生一些变化, 区别是静态类型的变化仅仅在使用时发生, 变量本身的静态类型不会被改变, 并且最终的静态类型是在编译期可知的; 而实际类型变化的结果在运行期才可确定, 编译器在编译程序的时候并不知道一个对象的实际类型是什么
```Java
// 实际类型变化, 在运行期才可知
Human man = new Man();
Human woman = new Woman();
// 静态类型变化
sd.sayHello((Man) man);
sd.sayHello((Woman) woman);
```
main() 中两次 sayHello() 方法调用, 在方法接收者已经确定的情况下, 使用哪个重载版本就完全取决于传入参数的数量和数据类型; 代码中可以定义了两个静态类型相同但是实际类型不同的变量, 但虚拟机 (准确的应该说编译器) 在重载时是通过参数静态类型而不是实际类型作为判定依据的; 并且静态类型是编译器可知的, Javac 编译器会根据参数的静态类型决定使用哪个重载版本, 所以选择了 sayHello(Human) 作为调用目标, 并把这个方法的符号引用写到 main() 方法里的两条 invokevirtual 指令的参数中  
所有依赖静态类型来定位方法执行版本的分派动作称为静态分派; 静态分派的典型应用是方法重载, 静态分派发生在编译阶段, 因此确定静态分派的动作实际上不是由虚拟机来执行的; 另外, 编译器虽然能确定出方法的重载版本, 但在很多情况下这个重载的版本并不是 "唯一的", 往往只能确定一个 "更合适的" 版本; 产生这种模糊结论的主要原因是字面量不需要定义, 所以字面量没有显式的静态类型, 它的静态类型只能通过语言上的规则去理解和推断
```Java
public class OverLoad {

    public static void sayHello(Object obj) {
        System.out.println("Hello, Object!");
    }

    public static void sayHello(int arg) {
        System.out.println("Hello, int!");
    }

    public static void sayHello(long arg) {
        System.out.println("Hello, long!");
    }

    public static void sayHello(Character arg) {
        System.out.println("Hello, Character!");
    }

    public static void sayHello(char arg) {
        System.out.println("Hello, char!");
    }

    /**
     * 基本类型和包装类型的数组参数会冲突
     * @param arg
     */
    public static void sayHello(char... arg) {
        System.out.println("Hello, char...!");
    }

    /**
     * Character 的同级接口会冲突
     * @param arg
     */
    public static void sayHello(Serializable arg) {
        System.out.println("Hello, Serializable!");
    }

    public static void main(String[] args) {
        sayHello('a');
    }

}
```
重载方法匹配优先级为: `char -> int -> long -> [float -> double ->] Character -> Serializable|Comparable<Character> -> Object -> char...|Character...`   
首先 'a' 是一个 char 类型, 其次 'a' 也可以代表数字 97, 所以也可以发生自动类型转换, 去匹配 int 等参数, 但是不能自动类型转换为 boolean, byte 和 short 等不兼容的转换; 当去掉所有基本类型参数的方法, 'a' 会发生自动装箱, 匹配 Character 参数的方法; 继续匹配实现接口的的方法, 同级的实现接口的优先级是相同的; 继续则是继承父类, 如果有多层父类, 则优先级是重下往上优先级递减的, 即使传入的参数值是 null 时, 这个规则仍然适用; 再继续则是 'a' 基本类型数组和包装类型数组, 这两个优先级是相同的  
以上代码演示了编译期间选择静态分派目标的过程, 这个过程也是 Java 语言实现方法重载的本质; 演示代码属于极端的例子, 除了用做面试题为难求职者以外, 在实际工作中几乎不可能有实际用途, 大部分这样极端的重载研究都可算是真正的 "关于茴香豆的茴有几种写法的研究"; 另外, 解析和分派这两者之间的关系并不是二选一的排他关系, 它们是在不同的层次上去筛选确定目标方法的过程; 静态方法会在类记载期就会进行解析, 而静态方法显然也使可以拥有重载版本的, 选择重载版本的过程也是通过静态分派完成的  

###### 动态分派
```Java
public class DynamicDispatch {

    static abstract class Human {
        protected abstract void sayHello();
    }

    static class Man extends Human {
        @Override
        protected void sayHello() {
            System.out.println("Man Say Hello!");
        }
    }

    static class Woman extends Human {
        @Override
        protected void sayHello() {
            System.out.println("Woman Say Hello!");
        }
    }

    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        man.sayHello();
        woman.sayHello();
        man = new Woman();
        man.sayHello();
    }

}
```
动态分派和多态性的另外一个体现 --- 重写有着很密切的关联; 以上方法调用显然不能再根据静态类型来确定, javap 字节码的找到方法调用的 invokevirtual 指令, 原因就需要从 invokevirtual 指令的多态查找过程说起; invokevirtual 指令的运行时解析过程大致分为以下几个步骤
- 找到操作数栈顶的第一个元素所指向的对象的实际类型, 记作 C
- 如果在类型 C 中找到与常量中描述符和简单名称都相符的方法, 则进行访问权限校验, 如果通过则返回这个方法的直接引用, 查找过程结束; 如果不通过, 则返回 java.lang.IllegalAccessError 异常
- 否则, 按照继承关系从下往上依次对 C 的各个父类进行第 2 步的搜索和验证过程
- 如果始终没有找到合适的方法, 则抛出 java.lang.IllegalAccessError 异常

由于 invokevirtual 指令执行的第一步就是在运行期确定接受者的实际类型, 所以两次调用中的 invokevirtual 指令把常量池中的类方法引用解析到了不同的直接引用上, 这个过程就是 Java 语言中方法重写的本质; 这种在运行期根据实际类型确定方法执行版本的分派过程称为动态分派

###### 单分派与多分派
方法的接收者和方法的参数统称为方法的宗量, 根据分派基于多少种宗量, 可以将分派划分为单分派和多分派两种; 单分派是根据一个宗量对目标方法进行选择, 多分派则是根据多于一个宗量对目标方法进行选择
```Java
public class Dispatch {

    static class QQ {}

    static class _360 {}

    public static class Father {

        public void hardChoice(QQ arg) {
            System.out.println("Father choose qq!");
        }

        public void hardChoice(_360 arg) {
            System.out.println("Father choose 360!");
        }
    }

    public static class Son extends Father {

        @Override
        public void hardChoice(QQ arg) {
            System.out.println("Father choose qq!");
        }

        @Override
        public void hardChoice(_360 arg) {
            System.out.println("Father choose 360!");
        }
    }

    public static void main(String[] args) {
        Father father = new Father();
        Father son = new Son();
        father.hardChoice(new _360());
        son.hardChoice(new QQ());
    }

}
```
编译阶段编译器的选择过程, 也就是静态分派的过程; 这时选择目标方法的依据有两点: 一是静态类型是 Father 和 Son, 二是方法参数是 QQ 还是 360; 这次选择结果的最终产物是产生了两条 invokevirtual 指令, 两条指令的参数分别为常量池种指向 Father.hardChoice(360) 和 Father.hardChoice(QQ) 方法的符号引用; 因为是根据两个宗量进行选择, 所以 Java 语言的静态分派属于多分派类型  
运行阶段虚拟机的选择, 也就是动态分派的过程; 在执行 son.hardChoice(new QQ()) 代码时, 更准确的说是执行对应的 invokevirtual 指令时; 由于编译期已经决定了目标方法的签名必须为 hardChoice(QQ), 虚拟机此时不会关心传递过来的参数到底是什么, 因为这时参数的静态类型和实际类型对方法的选择都不会构成任何影响, 唯一可以影响虚拟机选择的因素只有此方法的接收者的实际类型是 Father 还是 Son; 因为只有一个宗量作为选择依据, 所以 Java 语言的动态分派属于单分派类型  
所以可以总结为: Java 语言是一门静态多分派, 动态单分派的语言

###### 虚拟机动态分派的实现
由于动作分派是非常频繁的动作, 而且动作分派的方法版本选择过程需要运行时在类的方法元数据种搜索合适的目标方法, 因此在虚拟机的实际实现中基于性能的考虑, 大部分实现都不会真正地进行如此频繁的搜索; 最常用的 "稳定优化" 手段就是为类在方法区中建立一个虚方法表 (Virtual Method Table, 也称 vtable; 于此对应的, 在 invokeinterface 执行时也会用到接口方法表 Interface Method Table, 也称 itable), 使用虚方法表索引来代替元数据查找以提高性能  
虚方法表中存放着各个方法的实际入口地址, 如果某方法在子类中没有被重写, 那子类的虚方法表里面的地址入口和父类相同地方的地址入口是一致的, 都指向父类的实现入口; 如果子类中重写了这个方法, 子类方法表中的地址将会替换为子类实现版本的入口地址; 为了程序实现上的方便, 具有相同签名的方法, 在父类和子类的虚方法表中都应当具有一样的索引号, 这样当类型变换时仅需要变更查找的方法表, 就可以从不同的虚方法表中按索引转换出所需的入口地址; 方法表一般在类加载的连接阶段进行初始化, 准备了类的变量初始值后, 虚拟机会把该类的方法表也初始化完毕  
方法表时分派调用的 "稳定优化" 手段, 虚拟机除了使用方法变表之外, 在条件允许的情况下, 还会使用内联缓存 (Inline Cache) 和基于 "类型继承关系分析" (Class Hierarchy Analysis, CHA) 技术的守护内联 (Guarded Inlinging) 两种非稳定的 "激进优化" 手段来获得更高的性能

##### 动态类型语言支持
随着 JDK 7 的发布, 字节码指令集终于迎来了第一位新成员 --- invokedynamic 指令; 这条新增加的指令是 JDK 7 实现 "动态类型语言" (Dynamically Typed Language) 支持而进行的改进之一, 也是为 JDK 8 可以顺利实现 Lambda 表达式做技术准备

###### 动态类型语言
动态类型语言的关键特征是它的类型检查的主体过程是在运行期而不是编译期, 满足这个特征的语言有很多, 常用的包括: APL, Clojure, Erlang, Groovy, JavaScript, Jython, Lisp, Lua, PHP, Prolog, Python, Ruby, Smalltalk 和 Tcl 等; 相对的, 在编译期就进行类型检查过程的语言 (如 C++ 和 Java 等) 就是最常用的静态类型语言   
"变量无类型而变量值才有类型" 也是动态类型语言的一个重要特征, 动态语言在编译时最多只能确定方法名称, 参数, 返回值等这些信息, 而不会确定方法所在的具体类型 (即方法的接受者不确定)  

###### JDK 7 与动态类型
Java 虚拟机层面对动态类型语言的支持一直都有所欠缺, 主要表现在方法调用方面: JDK 7 之前的字节码指令之中, 4 条方法调用指令 (invokevirtual, invokespecial, invokestatic, invokeinterface) 的第一个参数都是被调用方法的符号引用 (CONSTANT_Methodref_info 或者 CONSTANT_InterfaceMethodref_info 常量); 方法的符号引用在编译时产生, 而动态类型语言只有在运行期才能确定接收者类型, 所以在 Java 虚拟机上实现的动态类型语言就不得不使用其他方式 (如在编译时留个占位符类型, 运行时动态生成字节码实现具体类型到占位符类型的适配) 来实现, 这样势必让动态类型语言实现的复杂度增加, 也可能带来额外的性能或内存开销; 但这种低层问题终归是应当在虚拟机层次上去解决才最合适, 这也就是 JDK 7 (JSR-292) 中 invokedynamic 指令以及 java.lang.invoke 包出现的技术背景

###### java.lang.invoke 包
这个包的主要目的是在之前单纯依靠符号引用来确定调用的目标方法这种方式之外, 提供一种新的动态确定目标方法的机制, 称为 MethodHandle
```Java
public class MethodHandleTest {

    static class ClassA {
        public void println(String str) {
            System.out.println(str);
        }
    }

    public static void main(String[] args) throws Throwable {

        Object obj = System.currentTimeMillis() % 2 == 0 ? System.out : new ClassA();

        // 无论 obj 最终是哪个实现类, 这句代码都能正确的调用到 println 方法
        getPrintlnMH(obj).invokeExact("ltchen");

    }

    private static MethodHandle getPrintlnMH(Object receiver) throws NoSuchMethodException, IllegalAccessException {
        /*
         * MethodType: 代表 "方法类型", 包含了方法的返回值 (methodType() 的第一参数) 和具体参数 (methodType() 第二个及以后的参数)
         */
        MethodType mt = MethodType.methodType(void.class, String.class);

        /*
         * MethodHandles.lookup(): 是在指定类中查找符合给定的方法名称, 方法类型, 并且符合调用权限的方法句柄
         * 因为这里调用的是一个虚方法, 按照 Java 语言的规则, 方法的第一个参数是隐式的, 代表该方法的接收者, 也是 this 指向的对象, 这个参数以前
         * 是放在参数列表中进行传递的, 而现在提供了 bingTo() 方法完成这件事情
         */
        return MethodHandles.lookup().findVirtual(receiver.getClass(), "println", mt).bindTo(receiver);
    }

}
```
方法 getPrintlnMH() 中模拟了 invokevirtual 指令的执行过程, 只不过它的分派逻辑并非固化在 Class 文件的字节码上, 而是通过一个具体方法来实现; 而这个方法本身的返回值 (MethodHandle 对象), 可以视为对最终调用方法的一个 "引用"  
以上事情也可以用反射来实现, 如果仅站在 Java 语言的角度来看, MethodHandle 的使用方法和效果与 Reflection 有众多相似之处, 但是仍有以下区别
- 本质上讲, Reflection 和 MethodHandle 机制都是在模拟方法调用, 但 Reflection 是在模拟 Java 代码层次的方法调用, 而 MethodHandle 是在模拟字节码层次的方法调用; 在 MethodHandles.lookup 中的 3 个方法中 --- findStatic(), findVirtual(), findSpecial() 正是为了对应 invokestatic, invokevirtual & invokeinterface, invokestatic 这几条字节码指令的执行权限校验行为; 而这些低层细节在使用 Reflection API 时是不需要关心的
- Reflection 中的 java.lang.reflect.Method 对象远比 MethodHandle 机制中的 java.lang.invoke.MethodHandle 对象所包含的信息多; 前者是方法在 Java 端的全面映像, 包含了方法的签名, 描述符以及方法属性表中各种属性的 Java 端表示, 还包含执行权限等的运行期信息; 而后者仅仅包含与执行该方法相关的信息; 通俗的讲, Reflection 是重量级的, 而 MethodHandle 是轻量级的
- 由于 MethodHandle 是对字节码的方法指令调用的模拟, 所以理论上虚拟机在这方面做的各种优化 (如方法内联), 在 MethodHandle 上也应当可以采用类似的思路去支持, 而通过反射去调用方法则不行

最关键的一点还在于去掉前提 "仅站在 Java 语言的角度去看": Reflection API 的设计目标只为了 Java 语言服务的, 而 MethodHandle 则设计成可服务于所有 Java 虚拟机之上的语言, 其中也包括 Java 语言

###### invokedynamic 指令
在某种程度上, invokedynamic 指令与 MethodHandle 机制的作用是一样的, 都是为了解决原有 4 条 `invoke*` 指令方法分派规则固化在虚拟机之中的问题, 把如何查找目标方法的决定权从虚拟机转嫁到具体用户代码之中, 让用户 (包含其他语言的设计这) 有更高的自由度; 它们两者的思路也是可类比的, 可以把它们想象成为了达到同一个目的, 一个采用上层 Java 代码和 API 实现, 另一个用字节码和 Class 中其他属性, 常量完成

TODO...

###### 掌握方法分派规则
`invokedynamic` 指令与前面 4 条 `invoke*` 指令的最大差别就是它的分派逻辑不是由虚拟机决定的, 而是由程序员决定的
```Java
class GrandFather {
    void thinking() {
        System.out.println("I am grandfather");
    }
}

class Father extends GrandFather {
    void thinking() {
        System.out.println("I am father");
    }
}

class Son extends Father {
    void thinking() {
        // 在这里填入适当的代码, 实现调用祖父类的 thinking() 方法, 打印 "I am grandfather"
    }
}
```
在 Java 程序中, 可以通过 `super` 关键字很方便的调用到父类中的方法, 但是要访问祖父类的方法在 JDK 1.7 之前使用纯粹的 Java 语言很难处理这个问题 (直接生成字节码就很简单, 如使用 ASM 等字节码工具), 原因是在 `Son` 类的 `thinking()` 方法中无法获取一个实际类型是 `GrandFather` 的对象引用, 而 `invokevirtual` 指令的分派逻辑就是按照方法接收者的实际类型进行分派的, 但这个逻辑固化在虚拟机中, 程序员无法改变; 在 JDK 1.7 及以后可以使用以下代码解决这个问题
```Java
class GrandFather {
    void thinking() {
        System.out.println("I am grandfather");
    }
}

class Father extends GrandFather {
    void thinking() {
        System.out.println("I am father");
    }
}

class Son extends Father {
    void thinking() {
        try {
            MethodType type = MethodType.methodType(void.class);
            // MethodHandle handle = MethodHandles.lookup().findSpecial(GrandFather.class, "thinking", type, getClass());
            // see https://www.zhihu.com/question/40427344/answer/86545388
            Field IMPL_LOOKUP = MethodHandles.Lookup.class.getDeclaredField("IMPL_LOOKUP");
            IMPL_LOOKUP.setAccessible(true);
            MethodHandles.Lookup lookup = (MethodHandles.Lookup) IMPL_LOOKUP.get(null);
            MethodHandle handle = lookup.findSpecial(GrandFather.class, "thinking", type, GrandFather.class);
            handle.invoke(this);
        } catch (Throwable e) {
            System.out.println(e.getMessage());
        }
    }

    public static void main(String[] args) {
        new Son().thinking();
    }
}
```

#### 基于栈的字节码解释执行引擎
许多 Java 虚拟机的执行引擎在执行 Java 代码的时候都有解释执行 (通过解释器执行) 和编译执行 (通过即时编译器产生本地代码执行) 两种选择

##### 解释执行
Java 语言常被人定位为 "解释执行" 语言, 在 Java 初始的 1.0 时代, 这种定义还算比较准确的, 但当主流的虚拟机中都包含了即时编译器后, Class 文件中的代码到底会被解释执行还是编译执行, 就成了只有虚拟机自己才能准确判断的事; 再后来, Java 发展出了可以直接生成本地代码的编译器 (如 GCJ, GNU Compiler for the java), 而 C/C++ 语言也出现了通过解释器执行的版本 (如 CINT); 这时候再笼统的说 "解释执行", 对整个 Java 语言来说就成了没有意义的概念, 只有确定了谈论对象是某种具体的 Java 实现版本和执行引擎运行模式时, 谈论解释执行还是编译执行才会比较确切  
大部分的程序代码到物理机的目标代码或虚拟机能执行的指令集之前, 都需要经过以下几个步骤; 上面这条分支就是解释执行的过程, 下面那条分支就是传统编译原理中程序代码到目标机器代码的生成过程
```
                                                      指令流 (可选) -> 解释器 -> 解释执行
                                                      /
程序源码 -> 词法分析 -> 单词流 -> 语法分析 -> 抽象语法树
                                                      \
                                                      优化器 (可选) -> 中间代码 (可选) -> 生成器 -> 目标代码
```
对于一门具体语言实现来说, 词法分析, 语法分析已至后面的优化器和目标代码生成器都可以选择独立于执行引擎, 形成一个完整意义的编译器去实现, 这类代表是 C/C++ 语言; 也可以选择把其中一部分步骤 (如生成抽象语法树之前的步骤) 实现为一个半独立的编译器, 这类代表是 Java 语言; 又或者把这些步骤和执行引擎全部集中封装在一个封闭的黑匣子中, 如大多数的 JavaScript 执行器  
Java 语言中, Javac 编译器完成了程序代码经过词法分析, 语法分析到抽象语法树, 再遍历语法树生成线性的字节码指令流的过程; 因为这一部分动作是在 Java 虚拟机之外进行的, 而解释器在虚拟机内部, 所以 Java 程序的编译就是半独立的实现

##### 基于栈的指令集与基于寄存器的指令集
Java 编译器输出的指令流, 基本上是一种基于栈的指令集架构 (Instruction Set Architecture, ISA), 指令流中的指令大部分都是零地址指令, 它们依赖操作数栈进行工作; 与之相对的另外一套常用的指令集架构是基于寄存器的指令集, 最典型的就是 x86 的二地址指令, 即主流 PC 机中直接支持的指令集架构; 那么基于栈的指令集与基于寄存器的指令集的这两者之间有什么不同呢?  
基于栈的指令集主要优点是可移植, 寄存器由硬件直接提供, 程序直接依赖这些硬件寄存器则不可避免的要受到硬件的约束; 还有就是代码相对更加紧凑 (字节码中的每个字节都对应一条指令, 而多地址指令集中还需要存放参数), 编译器实现更加简单 (不需要考虑空间分配的问题, 所需空间都在栈上操作) 等  
栈的指令集的主要缺点是执行速度相对来说会稍慢一些; 虽然栈架构的指令集的代码非常紧凑, 但是完成相同功能所需的指令数量一般会比寄存器架构多, 因为出栈入栈操作本身就产生了相当多的指令数量; 更重要的是, 栈实现在内存之中, 频繁的栈访问也就意味着频繁的内存访问, 相对于处理器来说, 内存始终是执行速度的瓶颈; 尽管虚拟机可以采取栈顶缓存的手段, 把最常用的操作映射到寄存器中避免直接内存访问, 但这也只能是优化措施而不是解决本质问题的方法, 由于指令数量和内存访问的原因, 所以导致了栈架构指令集执行速度会相对缓慢

##### 基于栈的解释器执行过程
TODO...
