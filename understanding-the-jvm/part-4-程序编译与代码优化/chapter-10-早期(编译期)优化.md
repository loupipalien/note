### 早期 (编译期) 优化
从计算机程序出现的第一天起, 对效率的追求就是程序天生的坚定信仰, 这个过程犹如一场没有终点, 永不停歇的 F1 方程式竞赛, 程序员是车手, 技术平台则是赛道上飞驰的赛车

#### 概述
Java 语言的 "编译期" 其实是一段 "不确定" 的操作过程, 因为它可能是指一个前端编译器把 `*.java` 文件转变为 `*.class` 文件的过程; 也可能是值虚拟机的后端运行期编译器 (JIT 编译器, Just In Time Compiler) 把字节码变成机器码的过程; 还有可能是只使用静态提前编译器 (AOT 编译器, Ahead Of Time Compiler) 直接把 `*.java` 文件编译成本地机器代码的过程; 以下是这 3 类编译过程中一些比较有代表性的编译器
- 前端编译器: Sun 的 Javac, Eclipse JDT 中的增量式编译器 (ECJ)
- JIT 编译器: HotSpot VM 的 C1, C2 编译器
- AOT 编译器: GNU Compiler for the Java (GCJ), Excelsior JET

Javac 这类编译器对代码的运行效率几乎没有任何优化措施 (JDK 1.3 之后, Javac 的 -O 优化参数就不再有意义); 虚拟机设计团队把对性能的优化集中到了后端的即时编译器中, 这样可以让哪些不是由 Javac 产生的 Class 文件 (如 JRuby, Groovy 等语言的 Class 文件) 也同样能享受到编译器优化带来的好处; 但是 Javac 做了许多针对 Java 语言编码过程的优化措施来改善程序员的编码风格和提高编码效率; 相当多新生的 Java 语法特性, 都是靠编译器的 "语法糖" 来实现, 而不是依赖虚拟机的底层改进支持; 可以说, Java 中的即时编译器在运行期的优化过程对于程序运行来说更重要, 而前端编译器在编译期的优化过程对于程序编码来说关系更加密切  

#### Javac 编译器
Javac 编译器是由 Java 语言编写的程序

##### Javac 的源码与调试
TODO...

##### 解析和填充符号表
解析步骤由 `com.sun.tools.javac.main.JavaComplier.parseFiles()` 方法完成, 解析步骤包括了词法分析和语法分析两个过程
- 词法, 语法分析
词法分析是将源代码中的字符流转变为标记 (Token) 集合, 单个字符是程序编写过程中的最小元素, 而标记则是编译过程中的最小元素; 关键字, 变量名, 字面量, 运算符都可以成为标记; 在 Javac 的源码中, 词法分析过程是由 `com.sun.tools.javac.parser.Scanner` 类来实现的
语法分析是根据 Token 序列构造抽象语法树的过程, 抽象语法树 (Abstract Synatax Tree, AST) 是一种用来描述程序代码语法结构的树形表示方式, 语法树的每一个节点都代表着程序代码中的一个语法结构 (Construct), 例如包, 类型, 修饰符, 运算符, 接口, 返回值至代码注释等都可以是一个语法结构; 在 Javac 的源码中, 语法分析过程是由 `com.sun.tools.javac.parser.Parser` 类实现的, 这个阶段产生的抽象语法树由 `com.sun.tools.javac.tree.JCTree` 类表示
- 填充符号表
符号表 (Symbol Table) 是由一组符号地址和符号信息构成的表格, 符号表中所登记的信息在编译的不同阶段都要用到; 在语义分析中, 符号表所登记的内容将用于语义检查和产生中间代码, 在目标代码生成阶段, 当对符号名进行地址分配时, 符号表是地址分配的依据; 在 Javac 源代码中, 由 `com.sun.tools.javac.main.JavaComplier.enterTree()` 方法触发, 填充符号表的过程由 `com.sun.tools.javac.comp.Enter` 类实现, 此过程的出口是一个待处理列表 (To Do List), 包含了每一个编译单元的抽象语法树的顶级节点以及 package-info.java (如果存在的话) 的顶级节点

##### 注解处理器
JDK 1.5 之后, Java 语言提供了对注解 (Annotation) 的支持, 这些注解与普通的 Java 代码一样, 是在运行期间发挥作用的; 在 JDK 1.6 中实现了 JSR-269 规范, 提供了一组插入式注解处理器的标准 API 在编译期间对注解进行处理, 可以把它看作是一组编译器的插件, 在这些插件里面, 可以读取, 修改, 添加抽象语法树中的任意元素; 如果这些插件在处理注解期间对语法树进行了修改, 编译器将回到解析及填充符号表的过程重新处理, 直到所有插入式注解处理器都没有再对语法树进行修改为止, 每一次循环称为一个 Round  
在 Javac 源码中, 插入式注解处理器的初始化过程是在  `com.sun.tools.javac.main.JavaComplier.initProccessAnnotation()` 方法中完成的, 而它的执行过程则是在 proccessAnnotation() 方法中完成的, 这个方法判断是否还有新的注解处理器需要执行, 如果有的话, 通过 `com.sun.tools.javac.processing.JavacProcessingEnvironment` 类的 doProcessing() 方法生成一个 JavaComplier 对象对编译的后续步骤处理

##### 语义分析与字节码生成
语法分析之后, 编译器获得了程序代码的抽象语法树表示, 语法树能表示一个结构正确的源程序的抽象, 但无法保证源程序是符合逻辑的; 而语义分析的主要任务是对结构上正确的源程序进行上下文有关性质的审查, 如进行类型审查等
- 标注检查
Javac 的编译过程中, 语义分析过程分为标注检查以及数据及控制流分析两个步骤, 分别由 `com.sun.tools.javac.main.JavaComplier.attribute()` 和 `com.sun.tools.javac.main.JavaComplier.flow()` 方法完成; 标注检查步骤检查的内容包括变量使用前是否已被声明, 变量与赋值之间的数据类型是否能够匹配等; 标注检查步骤在 Javac 源码中的实现类是 `com.sun.tools.javac.comp.Attr` 和 `com.sun.tools.javac.comp.Check` 类
- 数据及控制流分析
数据及控制流分析是对程序上下文逻辑更进一步的验证, 它可以检查出程序局部变量在使用前是否赋值, 方法的每条路径是否都有返回值, 是否所有的受查异常都被正确处理了等问题; 编译时期的数据及控制流分析与类加载时的数据及控制流分析的目的基本上是一致的, 但是校验范围有所区别, 有些校验项只有在编译期或运行期才能进行; 在 Javac 的源码中, 数据及控制流分析的入口是 `com.sun.tools.javac.main.JavaComplier.flow()`, 实现是 `com.sun.tools.javac.comp.Flow` 类
```Java
public void foo(final int arg) {
    final int var = 0;
    // ...
}

public void foo(int arg) {
    int var = 0;
    // ...
}

/*
局部变量与字段 (实例变量, 类变量) 是有区别的, 它在常量池中没有 CONSTANT_Fieldref_info 的符号引用, 自然就没有访问标志的信息, 甚至可能连名称都不会保留下来 (取决于编译时的选项), 自然在 Class 文件中不可能知道局部变量是不是声明为 final 了; 因此, 将局部变量声明为 final, 对运行期是没有影响的, 变量的不变性仅仅由编译器在编译期间保障
*/
```
- 解语法糖
语法糖 (Syntactic Sugar), 是由英国计算机科学家皮得.约翰.兰达 (Peter J.Landin) 发明的一个术语, 指在计算机语言中添加的某种语法, 这种语法对语言的功能并没有影响, 但更方便程序员使用; 通常来说, 使用语法糖能够增加程序的可读性, 从而减少程序代码出错的机会; 在 Javac 的源码中, 解语法糖的过程由 `com.sun.tools.javac.main.JavaComplier.desugar()` 触发, 在 `com.sun.tools.javac.comp.TransTypes` 和 `com.sun.tools.javac.comp.Lower` 类中完成

##### 字节码生成
字节码生成是 Javac 编译过程的最后一个阶段, 在 Javac 源码里由 `com.sun.tools.javac.jvm.Gen` 类来完成, 字节码生成阶段不仅仅是把前面各个步骤所生成的信息 (语法树, 符号表) 转化成字节码写到磁盘中, 编译器还进行了少量的代码添加和转换工作  
实例构造器 `<init>()` 方法和类构造器 `<clinit>()` 方法就是在这个阶段添加到语法树当中的 (注意, 这里的实例构造器并不是指默认构造函数, 如果用户代码中没有提供任何构造函数, 那编译器将会添加一个没有参数的, 访问性和当前类一致的默认构造函数, 这个工作在填充符号表阶段就已经完成), 这两个构造器的产生过程实际上是一个代码收敛的过程, 编译器会把语句块 (对于实例构造器而言是 "{}" 块, 对于类构造器而言是 "static{}" 块), 变量初始化 (实例变量和类变量), 调用父类的实例构造器 (仅仅是实例构造器, `<clinit>()` 方法中无须调用父类的 `<clinit>()` 方法, 虚拟机会自动保证父类构造器的执行) 等操作收敛到 `<init>()` 和 `<clinit>()` 方法之中, 并且保证一定是按先执行父类的实例构造器, 然后初始化变量, 最后执行语句块的顺序进行; 除了生成构造器之外, 还有其他的一些代码替换工作用于优化程序的实现逻辑  
完成了对语法树的遍历和调整之后, 就会把填充了所需信息的符号表交给 `com.sun.toools.javac.jvm.ClassWriter` 类, 由这个类的 `writeClass()` 方法输出字节码, 生成最终的 Class 文件, 到此为止整个编译过程宣告结束

#### Java 语法糖的味道
语法糖可以看做是编译器实现的一些 "小把戏", 这些 "小把戏" 可能会使效率 "大提升"

##### 泛型与类型擦除
泛型是 JDK 1.5 的一项新增特性, 它的本质是参数化类型 (Parameterisized Type) 的应用, 也就是说所操作的数据类型被指定为一个参数; 这种参数类型可以用在类, 接口和方法的创建中, 分别称为泛型类, 泛型接口, 泛型方法   
泛型技术在 C# 和 Java 之中的使用方式看似相同, 但实现上却有着根本性的分歧; C# 里泛型无论在程序源码中, 编译后的 IL 中 (Intermediate Language, 中间语言, 这时候泛型是一个占位符), 或是运行期的 CLR 中, 都是切实存在的, `List<int>` 和 `List<String>` 就是两种不同的类型, 它们在系统运行期生成, 有自己的虚方法表和类型数据, 这种实现称为类型膨胀, 基于这种方法实现的泛型称为真实泛型  
Java 语言中的泛型则不一样, 它只在源程序中存在, 在编译后的字节码文件中, 就已经替换为原来的原生类型 (Raw Type, 也称裸类型) 了, 并且在相应的地方插入了强制转换代码; 因此, 对于运行期的 Java 语言来说, `ArrayList<int>` 和 `ArrayList<String>` 就是同一个类, 所以泛型技术实际上是 Java 语言的一颗语法糖; Java 语言中的泛型实现方法称为类型擦除, 基于这种方法实现的泛型称为伪泛型  
JCP 组织对虚拟机规范做出了相应修改, 引入了诸如 `Signature, LocalVariableTypeTable` 等新的属性用于解决伴随泛型而来的参数类型的识别问题, `Signature` 是其中最重要的一项属性, 它的作用就是存储一个方法在字节码层面的特征签名, 这个属性中保存的参数类型并不是原生类型, 而是包括了参数化了类型的信息; 修改后的虚拟机规范要求所有能识别 49.0 以上版本的 Class 文件的虚拟机都要能正确的识别 `Signature` 参数

##### 自动装箱, 拆箱与遍历循环
包装类的 "==" 运算在不遇到算术运算的情况下不会自动拆箱, 以及它们 equals() 方法不处理数据类型的关系, 建议在实际代码中尽量避免这样使用自动装箱与拆箱
```
public static void main(String[] args) {
    // 自动装箱的陷阱
    Integer a = 1;
    Integer b = 2;
    Integer c = 3;
    Integer d = 3;
    Integer e = 321;
    Integer f = 321;
    Long g = 3L;
    System.out.println(c == d); // true, [-128, 127] 有缓存
    System.out.println(e == f); // false
    System.out.println(c == (a + b)); // true
    System.out.println(c.equals(a + b)); // true, [-128, 127] 有缓存
    System.out.println(g == (a + b)); // true
    System.out.println(g.equals(a + b)); // false
}
```

##### 条件编译
许多程序设计语言都提供了条件编译的途径, 如 C, C++ 中使用预处理器指示符 (#ifdef) 来完成条件编译; C, C++ 的预处理器最初的任务是解决编译时代码依赖关系 (如 #include 预处理命令), 而 Java 语言中并没有使用预处理器, 因为 Java 语言天然的编译方式 (编译器并非一个个地编译文件, 而是将所有的编译单元的语法树顶级节点输入到待处理列表后再进行编译, 因此各个文件之间能够互相提供符号信息) 无须使用预处理器  
但 Java 语言也能进行条件编译, 方法就是使用条件为常量的 if 语句; 以下代码中的 if 语句不同于其他 Java 代码, 它在编译阶段就会被 "运行", 生成的字节码之中只包括 `System.out.println("block 1");` 这一条语句
```
public static void main(String[] args) {
    // 条件编译
    if (true) {
        System.out.println("block 1");
    } else {
        System.out.println("block 1");
    }
}
```
只能使用条件为常量的 if 语句才能达到上述效果, 如果使用常量与其他带有条件判断能力的语句搭配, 则可能在控制流分析中提示错误, 被拒绝编译; Java 语言中条件编译的实现, 也是 Java 语言的一颗语法糖, 根据布尔常量值的真假, 编译器将会把分支中不成立的代码块消除掉, 这一工作将在编译器解除语法糖阶段 (`com.sun.tools.javac.comp.Lower`) 完成; 由于这种条件编译的实现方式使用了 if 语句, 所以它必须遵循最基本的 Java 语法, 只能写在方法内部, 因此它只能实现语句基本块级别的条件编译, 而没有办法实现根据条件调整整个 Java 类的结构

#### 实战: 插入式注解处理器
TODO...
