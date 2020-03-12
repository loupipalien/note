### 类文件结构
代码编译的结果从本地机器码转变为字节码, 是存储格式发展的一小步, 却是编程语言发展的一大步

#### 概述
计算机虽然只能识别 0 和 1, 当由于近十年来虚拟机以及大量建立在虚拟机上的程序语言如雨后春笋出现并蓬勃发展, 将编写的程序编译成二进制本地机器码 (Native Code) 已不再是唯一的选择, 越来越多的程序选择了与操作系统和机器指令集无关的, 平台中立的格式作为程序编译后的存储格式

#### 无关性基石  
- 虚拟机和字节码是构成平台无关性的基石
- 虚拟机和字节码也是实现语言无关性的基础

Java 虚拟机不和包括 Java 在内的任何语言绑定, 它只与 "Class 文件" 这种特定的二进制文件格式所关联, Class 文件中包含了 Java 虚拟机指令集和符号表以及若干其他辅助信息; 基于安全方面的考虑, Java 虚拟机规范要求在 Class 文件中使用许多强制性的语法和结构化约束, 但任何一门功能性语言都可以表示为一个能被 Java 虚拟机所接受的有效的 Class 文件; 作为一个通用的, 机器无关的执行平台, 任何其他语言的实现者都可以将 Java 虚拟机作为语言的产品交付媒介

#### Class 类文件的结构
Class 文件是一组以 8 位字节为基础单位的二进制流, 各个数据项目严格按照顺序紧凑的排列在 Class 文件中, 中间没有添加任何分隔符, 这使得整个 Class 文件中存储的内容几乎全部是程序运行的必要数据, 没有空隙存在; 当遇到需要占用 8 位字节以上空间的数据项时, 则会高位在前的方式分隔成若干个 8 位字节进行存储  
根据 Java 虚拟机规范的规定, Class　文件格式采用一种类似于 C 语言结构的伪结构来存储数据, 这种伪结构中只有两种数据类型: 无符号数和表, 后面的解析都要以这两种数据类型为基础
- 无符号数: 属于基本的数据类型, 以 u1, u2, u4, u8 来分别代表 1 个字节, 2 个字节, 4 个字节, 8 个字节的无符号数, 无符号数可以用来描述数字, 索引引用, 数量值或者按照 UTF-8 编码构成字符串值
- 表: 由多个无符号数或者其他表作为数据项构成的复合数据类型, 所有表都习惯性的以 "\_info" 结尾, 表用于描述有层次关系的复合结构的数据, 整个 Class 文件本质上就是一张表

Class 文件格式

|类型|名称|数量|  
|-|-|-|
|u4|magic|1|
|u2|minor_version|1|
|u2|major_version|1|
|u2|constant_pool_count|1|
|cp_info|constant_pool|constant_pool_count - 1|
|u2|access_flags|1|
|u2|this_class|1|
|u2|super_class|1|
|u2|interfaces_count|1|
|u2|interfaces|interfaces_count|
|u2|fields_count|1|
|field_info|fields|fields_count|
|u2|methods_count|1|
|method_info|methods|methods_count|
|u2|attributes_count|1|
|attribute_info|attributes|attributes_count|

无论是无符号数还是表, 当需要描述同一类但数量不定的多个数据时, 经常会使用一个前置的容量计数器加若干个连续的数据项的形式, 这时称这一系列连续的某一类型的数据为某以类型的集合

##### 魔数与 Class 文件的版本
每个 Class 文件的头 4 个字节称为魔数, 它的唯一作用是确定这个文件是否为一个能被虚拟机接受的 Class 文件, Class 文件的魔数值为: "0xCAFEBABE"  
紧接着魔数的 4 个字节存储的是 Class 文件的版本号: 第 5, 6 个字节是次版本号, 第 7, 8 个字节是主版本号; Java 版本号从 45 开始, JDK 1.1 之后每个大版本发布主版本号加 1, 高版本 JDK 兼容低版本的 Class 文件, 但不能运行以后版本的 Class 文件, 虚拟机也拒绝执行超过版本号的 Class 文件

##### 常量池
紧接着主次版本号之后的是常量池入口, 常量池可以理解为 Class 文件之中的资源仓库, 它是 Class 文件结构中与其他项目关联最多的数据类型,  也是占用 Class 文件空间最大的数据项目之一, 同时它还是在 Class 文件中第一个出现的表类型数据项目  
由于常量池中的常量数量不固定, 所以在常量池的入口需要放置一项 u2 类型的数据, 代表常量池容量计数值 (constant_pool_count); 这个容量计数不是从 1 开始而是从 0 开始, 在 Class 文件格式规范制定时, 设计者将 0 项常量空出来是有特殊考虑的, 这样做的目的在于满足后面某些指向常量池的索引值的数据在特定的情况下需要表达 "不引用任何一个常量池项目" 的含义, 这种情况就可以把索引值置为 0 来表示  
常量池中主要存放两大类常量: 字面量 (Literal) 和符号引用 (Symbolic References); 字面量比较接近 Java 语言层面的常量概念, 如文本字符串, 声明为 final 的常量值等; 而符号引用则属于编译原理方面的概念, 包括了下面三类常量:
- 类和接口的全限定名 (Fully Qualified Name)
- 字段的名称和描述符 (Descriptor)
- 方法的名称和描述符

Java 代码在进行 Javac 编译的时候, 并不像 C 和 C++ 那样有连接的步骤, 而是在虚拟机加载 Class 文件的时候进行动态连接; 也就是说 Class 文件中不会保存各个方法的, 字段的最终内存布局信息, 因此这些字段, 方法的符号引用不经过运行期转化的话就无法得到真正的内存入口地址, 也就无法直接被虚拟机使用; 当虚拟机运行时, 需要从常量池获得对应的符号引用, 再在类创建时或运行时解析, 翻译到具体的内存地址中  
常量池中的每一项常量都是一个表, 在 JDK 1.7 之前共有 11 种结构各不相同的表结构数据, 在 JDK 1.7 种为了更好的支持动态语言的调用, 又额外增加了 3 种 (CONSTANT_MethodHandle_info, CONSTANT_MethodType_info, CONSTANT_InvokeDynamic_info); 这 14 种表都有一个共同的特点, 就是表开始的第一位是一个 u1 类型的标志位 (tag), 代表当前这个常量属于哪种常量类型;常量池的项目类型如下:

|类型|标志|描述|
|-|-|-|
|CONSTANT_Utf8_info|1|UTF-8 编码的字段|
|CONSTANT_Integer_info|3|整型字面量|
|CONSTANT_Float_info|4|浮点型字面量|
|CONSTANT_Long_info|5|长整型字面量|
|CONSTANT_Double_info|6|双精度浮点型字面量|
|CONSTANT_Class_info|7|类或接口的符号引用|
|CONSTANT_String_ino|8|字符串类型字面量|
|CONSTANT_Fieldref_info|9|字段的符号引用|
|CONSTANT_Methodref_info|10|类中方法的符号引用|
|CONSTANT_InterfaceMethodref_info|11|接口中方法的符号引用|
|CONSTANT_NameAndType_info|12|字段或方法的部分符号引用|
|CONSTANT_MethodHandle_info|15|表示方法句柄|
|CONSTANT_MethodType_info|16|标识方法类型|
|CONSTANT_InvokeDynamic_info|18|表示动态方法调用点|

###### CONSTANT_Class_info 型常量的结构
|类型|名称|数量|
|-|-|-|
|u1|tag|1|
|u2|name_index|1|
其中 tag 是标志位, 用于区分常量类型; name_index 是一个索引值, 它指向常量池中一个 CONSTANT_Utf8_info 类型常量, 此常量代表了这类 (或者接口) 的全限定名

###### CONSTANT_Utf8_info 型常量的结构
|类型|名称|数量|
|-|-|-|
|u1|tag|1|
|u2|length|1|
|u1|bytes|length|
length 值说明了这个 UTF-8 编码的字符串是多少字节, 它后面紧跟着的长度为 length 字节的连续数据是一个使用 UTF-8 缩略编码表示的字符串; UTF-8 缩略编码与普通 UTF-8 编码的区别是: '\u0001' 到 '\u007f' 之间的字符串的缩略码使用一个字节表示, 从 '\u0080' 到 '\uo7ff' 之间的所有字符的缩略编码用两个字节表示, 从 '\0800' 到 '\ufff' 之间的所有字符的缩略编码就按照普通的 UTF-8 编码规则使用三个字节表示; 由于 Class 文件中方法, 字段等都需要引用 CONSTANT_Utf8_info 型常量来描述名称, 所以 CONSTANT_Utf8_info 型常量的最大长度也就是 Java 中方法, 字段名的最大长度, 即 u2 类型所能表示的最大值 65535 (64 KB)

###### 常量池中的 14 种常量项的结构总表
|常量|名称|类型|描述|
|-|-|-|-|
|CONSTANT_Utf8_info|tag|u1|值为 1|
|-|length|u2|UTF-8 编码的字符串占用的字节数|
|-|bytes|u1|长度为 length 的 UTF-8 编码的字符串|
|CONSTANT_Integer_info|tag|u1|值为 3|
|-|bytes|u4|按照高位在前存储 int 值|
|CONSTANT_Float_info|tag|u1|值为 4|
|-|bytes|u4|按照高位在前存储 float 值|
|CONSTANT_Long_info|tag|u1|值为 5|
|-|bytes|u8|按照高位在前存储 long 值|
|CONSTANT_Double_info|tag|u1|值为 6|
|-|bytes|u8|按照高位在前存储 double 值|
|CONSTANT_Class_info|tag|u1|值为 7|
|-|index|u2|指向全限定名常量项的索引|
|CONSTANT_Stringt_info|tag|u1|值为 8|
|-|bytes|u2|指向字符串字面量的索引|
|CONSTANT_Fieldref_info|tag|u1|值为 9|
|-|index|u2|指向声明字段的类或接口描述符 CONSTANT_Class_info 的索引项|
|-|index|u2|指向声明字段的类或接口描述符 CONSTANT_NameAndType_info 的索引项|
|CONSTANT_Methodref_info|tag|u1|值为 10|
|-|index|u2|指向声明字段的类或接口描述符 CONSTANT_Class_info 的索引项|
|-|index|u2|指向声明字段的类或接口描述符 CONSTANT_NameAndType_info 的索引项|
|CONSTANT_InterfaceMethodref_info|tag|u1|值为 11|
|-|index|u2|指向声明字段的类或接口描述符 CONSTANT_Class_info 的索引项|
|-|index|u2|指向声明字段的类或接口描述符 CONSTANT_NameAndType_info 的索引项|
|CONSTANT_NameAndType_info|tag|u1|值为 12|
|-|index|u2|指向该字段或方法名称常量项的索引|
|-|index|u2|指向该字段或方法描述符常量项的索引|
|CONSTANT_MethodHandle_info|tag|u1|值为 15|
|-|reference_kind|u1|值必须在 1 - 9 之间 (包括 1 和 9), 它决定了方法句柄的类型, 方法句柄类型的值表示方法句柄的字节码行为|
|-|reference_index|u2|值必须是对常量池的有效索引|
|CONSTANT_MethodType_info|tag|u1|值为 16|
|-|desctiptor_index|u2|值必须是对常量池的有效索引|
|CONSTANT_InvokeDynamic_info|tag|u1|值为 18|
|-|bootstrap_method_attr_index|u2|值必须是当前 Class 文件中引导方法 bootstrap_methods[] 数组的有效索引|
|-|name_and_type_index|u2|值必须是对当前常量池的有效索引, 常量池在该索引处的项必须是 CONSTANT_NameAndType_info 结构, 表示方法名和方法描述符|

##### 访问标志
在常量池结束之后, 紧接着的两个字节代表访问标志 (access_flags), 这个标志用于标识一些类或者接口层次的访问信息, 包括: 这个 Class 是类还是接口; 是否定于为 public 类型; 是否定义为 abstract 类型; 如果是类的话是否被声明为 final 等; 具体的标志位以及标志的含义见下表
|标志名称|标志值|含义|
|-|-|-|
|ACC_PUBLIC|0x0001|是否为 public|
|ACC_FINAL|0x0010|是否被声明为 final, 只有类可设置|
|ACC_SUPER|0x0020|是否允许使用 invokespecial 字节码指令的新语意, invokespecial 指令的语意在 JDK 1.0.2 发生过变化, 为了区别这条指令使用哪些语意, JDK 1.0.2 之后编译出来的类的这个标志都必须为真|
|ACC_INTERFACE|0x0200|标识这是一个接口|
|ACC_ABSTRACT|0x0400|是否为 abstract 类型, 对于接口或抽象类来说, 此标识值为真, 其他类值为假|
|ACC_SYNTHETIC|0x1000|标识这个类并非由用户代码产生的|
|ACC_ANNOTATION|0x2000|标识这是一个注解|
|ACC_ENUM|0x4000|标识这是一个枚举|
access_flags 中一共有 16 个标志位可用, 当前只定义了其中 8 个, 没有使用到的标志位要求一律为 0

##### 类索引, 父类索引与接口索引集合
类索引和父类索引都是一个 u2 类型的数据, 而接口索引集合是一组 u2 类型的数据集合, Class 文件中由这三项数据来确定这个类的继承关系; 类索引用于确定这个类的全限定名, 父类索引用于确定这个类的父类的全限定名; 由于 Java 不允许多重继承, 所以除了 java.lang.Object 之外, 所有 Java 类都有父类, 即父类索引都不为 0; 接口索引集合就用来描述这个类实现了哪些接口, 这些实现的接口将按 implements 语句 (如果这个类本身就是一个接口, 则应当是 extends 语句) 后的接口顺序从左到右排列在接口索引集合中  
类索引, 父类索引和接口索引集合都按照顺序排列在访问标志之后, 类索引和父类索引用两个 u2 类型的索引值表示, 它们各自指向一个类型为 CONSTANT_Class_info 的类描述符常量, 通过 CONSTANT_Class_info 类型的常量中的索引值就可以找到定义在 CONSTANT_Utf8_info 类型常量中的全限定名字符串  
对于接口集合, 入口的第一项 u2 类型的数据为接口计数器, 表示索引表的容量, 如果该类没有实现任何接口, 则该计数器值为 0, 后面的接口的缩影表不再占任何字节

##### 字段表集合
字段表 (field_info) 用于描述接口或类中的声明的变量; 字段包括类级变量以及实例变量, 但不包括在方法内部声明的局部变量; 字段可以包含的信息有: 字段的作用域 (public, private, protected 修饰符), 是实例变量还是类变量 (static 修饰符), 可变性 (final), 并发可见性 (volatile 修饰符, 是否强制从主内存读写), 可否被序列化 (transient 修饰符), 字段数据类型 (基本类型, 对象, 数组), 字段名称; 下表列出了字段表的最终格式
|类型|名称|数量|类型|名称|数量|
|-|-|-|-|-|-|
|u2|access_flags|1|u2|attributes_count|1|
|u2|name_index|1|attribute_info|attributes|attributes_count|
|u2|desctiptor_index|1|-|-|-|
字段修饰符放在 access_flags 中, 它与类中的 access_flags 是非常类似的, 都是一个 u2 的数据类型, 其中可以设置的标志位和含义见下表
|标志名称|标志值|含义|
|ACC_PUBLIC|0x0001|字段是否为 public|
|ACC_PRIVATE|0x0002|字段是否为 private|
|ACC_PROTECTED|0x0004|字段是否为 protected|
|ACC_STATIC|0X0008|字段是否为 static|
|ACC_FINAL|0x0010|字段是否为 final|
|ACC_VOLATILE|0x0040|字段是否为 volatile|
|ACC_TRANSIENT|0x0080|字段是否为 transient|
|ACC_SYNTHETIC|0x1000|字段是否由编译器自动产生的|
|ACC_ENUM|0x4000|字段是否为 enum|
在实际情况中, ACC_PUBLIC, ACC_PRIVATE, ACC_PROTECTED 三个标志最多只能选择其一, ACC_FINAL, ACC_VOLATILE 不能同时选择, 接口中字段必须有 ACC_PUBLIC, ACC_STATIC, ACC_FINAL 标志, 这些都是由 Java 本身语言规则所决定的  
跟随 access_flags 标志的是 name_index 和 desctiptor_index, 他们都是对常量池的引用, 分别代表着字段的简单名称以及字段和方法的描述符; 描述符的作用是用来描述字段的数据类型, 方法的参数列表 (包括数量, 类型以及顺序) 和返回值  
根据描述符的规则, 基本数据类型 (byte, char, double, float, int, long, short, boolean) 以及无返回值的 void 类型都用一个大写字符来表示, 而对象类型则用字符 L 加对象的全限定名来表示; 具体见下表
|标识字符|含义|标识字符|含义|
|-|-|-|-|
|B|基本类型 byte|J|基本类型 long|
|C|基本类型 char|S|基本类型 short|
|D|基本类型 double|Z|基本类型 boolean|
|F|基本类型 float|V|特殊类型 void|
|I|基本类型 int|L|对象类型, 如 Ljava/lan/Object|
对于数组类型, 每一个维度将使用一个前置的 "[" 字符来描述, 如一个定义为 "java.lang.String[]\[]" 类型的二维数组, 将被记录为: "[[Ljava/lang/String", 一个整型数组 "int[]" 将被记录为 "[I"  
用描述符来描述方法时, 按照先参数列表, 后返回值的顺序描述, 参数列表按照参数的严格顺序放在一组小括号 "()" 中; 如方法 void inc() 的描述符为 "()V" 方法 java.lang.String toString() 的描述符为 "()Ljava/lang/String", 方法 int indexOf(char[] source, int sourceOffset, int sourceCount, char[] target, int targetCount, int fromIndex) 的描述符为 "([CII[CIII)I"  
字段表集合中不会列出从超类或父接口中继承而来的字段, 但有可能列出原本 Java 代码中不存在的字段, 譬如在内部类中为了保持对外部类的访问性, 会自动添加指向外部类实例的字段; 另外在 Java 语言中字段时无法重载的, 两个字段的数据类型, 修饰符不管是否相同, 都必须使用不一样的名称, 但对于字节码来讲, 如果两个字段的描述符不一致, 那字段重名就是合法的  

##### 方法表集合
Class 文件存储格式中对方法的描述与对字段的描述几乎采用了完全一致的方式, 方法表的结构同字段表一样, 依次包括了访问标志, 名称索引, 描述符索引, 属性表集合几项; 具体见下表
|类型|名称|数量|类型|名称|数量|
|-|-|-|-|-|-|
|u2|access_flags|1|u2|attributes_count|1|
|u2|name_index|1|attribute_info|attributes|attributes_count|
|u2|desctiptor_index|1|-|-|-|
因为 volatile 关键字和 transient 关键字不能修饰方法, 所以方法表的访问标志中没有了 ACC_VOLATILE 和 ACC_TRANSIENT 标志; 与之相对的, synchronized, native, strictfp, abstract 关键字可以修饰方法, 所以方法表的访问标志中增加了 ACC_SYNCHRONIZED, ACC_NATIVE, ACC_STRICTFP, ACC_ABSTRACT 标志; 具体见下表
|标志名称|标志值|含义|
|ACC_PUBLIC|0x0001|方法是否为 public|
|ACC_PRIVATE|0x0002|方法是否为 private|
|ACC_PROTECTED|0x0004|方法是否为 protected|
|ACC_STATIC|0X0008|方法是否为 static|
|ACC_FINAL|0x0010|方法是否为 final|
|ACC_SYNCHRONIZED|0x0020|方法是否为 synchronized|
|ACC_BRIDGE|0x0040|方法是否是由编译器产生的桥接方法|
|ACC_VARARGS|0x0080|方法是否接受不定参数|
|ACC_NATIVE|0x0100|方法是否为 native|
|ACC_ABSTRACT|0x0400|方法是否为 abstract|
|ACC_STRICTFP|0x0800|方法是否为 strictfp|
|ACC_SYNTHETIC|0x1000|方法是否是由编译器自动产生的|
方法的定义可以通过访问标志, 名称索引, 描述符索引表表达清楚, 方法里的代码经过编译器编译成字节码指令后, 存放在方法属性表集合中一个名为 "Code" 的属性中  
与字段表集合相对应的, 如果父类方法在子类中没有被重写, 方法表集合中则不会出现来自父类的方法信息; 但同样的, 有可能会出现由编译器自动添加的方法, 最典型的便是类构造器 "<clinit>" 方法和实例构造器 "<init>" 方法  
在 Java 语言中, 要重载以一个方法, 除了要与原方法具有相同的简单名称之外, 还要求必须拥有一个与原方法不同的特征签名, 特征签名就是一个方法中各个参数在常量池中的字段符号引用的集合, 也因为返回值不会包含在特征签名中, 因此 Java 语言里是无法仅仅依靠返回值的不同来对一个已有方法进行重载的; 但在 Class 文件格式中, 特征签名的范围更大一些, 只要描述符不是完全一致的两个方法也可以共存; 也就是说如果两个方法有相同的名称和特征签名, 但是返回值不同, 那么也是可以合法共存于同一个 Class 文件中的

##### 属性表集合
在 Class 文件, 字段表, 方法表都可以携带自己的属性表集合, 以用于描述某些场景专有的信息  
与 Class 文件中其他的数据项目要求严格的顺序, 长度和内容不同, 属性表集合的限制稍微宽松了些, 不再要求各个属性表具有严格顺序, 并且只要不与已有属性名重复, 任何人实现的编译器都可以向属性表中写入自己定义的属性信息, Java 虚拟机运行时会忽略掉它不认识的属性, 在 <<Java 虚拟机规范 (Java SE 7)>> 版中, 预定义了 21 项虚拟机实现应当能识别的属性; 具体见下表
|属性名称|使用位置|含义|
|-|-|-|
|Code|方法表|Java 代码编译成的字节码指令|
|ConstantValue|字段表|final 关键字定义的常量值|
|Deprecated|类, 方法表, 字段表|被声明为 deprecated 的方法和字段|
|Exceptions|方法表|方法抛出的异常|
|EnclosingMethod|类文件|仅当一个类为局部类或匿名类时才能拥有这个属性, 这个属性用于标识这个类所在的外围方法|
|InnerClasses|类文件|内部类列表|
|LineNumberTable|Code 属性|Java 源码的行号与字节码指令的对应关系|
|LocalVariableTable|Code 属性|方法的局部变量描述|
|StackMapTable|Code 属性|JDK 1.6 中新增的属性, 供新的类型检查验证器 (Type Checker) 检查和处理目标方法的局部变量和操作数栈所需要的类型是否匹配|
|Signature|类, 方法表, 字段表|JDK 1.5 中新增的属性, 这个属性用于支持泛型情况下的方法签名, 在 Java 语言中, 任何类, 接口, 初始化方法或成员的泛型签名如果包含了类型变量 (Type Variables) 或参数化类型 (Parameterized Types), 则 Signature 属性会为它记录泛型签名信息; 由于 Java 的泛型采用擦除法实现, 在为了避免类型信息被擦除后导致签名混乱, 需要这个属性记录泛型中的相关信息|
|SourceFile|类文件|记录源文件名称|
|SourceDebugExtension|类文件|JDK 1.6 中新增的属性, SourceDebugExtension 属性用于存储额外的调试信息; 譬如在进行 JSP 文件调试时, 无法通过 Java 堆栈来定位到 JSP 文件的行号, JSR-45 规范为这些非 Java 语言编写, 却需要编译成字节码并运行在 Java 虚拟机中的程序提供了一个进行调试的标准机制, 使用 SourceDebugExtension 属性就可以用于存储这个标准所新加入的调试信息|
|Synthetic|类, 方法表, 字段表|标识方法或字段为编译器自动生成的|
|LocalVariableTypeTable|类|JDK 1.5 中新增的属性, 它使用特征签名代替描述符, 是为了引入泛型语法之后能描述泛型参数化类型而添加的|
|RuntimeVisibleAnnotions|类, 方法表, 字段表|JDK 1.5 中新增的属性, 为了动态注解提供支持; RuntimeVisibleAnnotions 属性用于指明哪些注解是运行是 (实际上运行时就是为了进行反射调用) 可见的|
|RuntimeInvisibleAnnotions|类, 方法表, 字段表|JDK 1.5 中新增的属性, 与 RuntimeVisibleAnnotions 属性作用相反, 用于指明哪些注解是运行时不可见的|
|RuntimeVisibleParameterAnnotions|方法表|JDK 1.5 中新增的属性, 作用与 RuntimeVisibleAnnotions 属性类似, 只不过作用于对象方法参数|
|RuntimeInvisibleParameterAnnotions|方法表|JDK 1.5 中新增的属性, 作用与 RuntimeInvisibleAnnotions 属性类似, 只不过作用于对象方法参数|
|AnnotationDefault|方法表|JDK 1.5 中新增的属性, 用于记录注解类元素的默认值|
|BootstrapMethods|类文件|JDK 1.7 中新增的属性, 用于保存 invokedynamic 指令引用的引导方法限定符|
对于每个属性, 它的名称需要从常量池中引用一个 CONSTANT_Utf8_info 类型的常量来表示, 而属性值的结构则是完全自定义的, 只需要通过一个 u4 的长度属性1去说明属性值所占用的位数即可; 一个符合规则的属性表应满足下表所定义的表结构

###### Code 属性
TODO...

#### 字节码指令简介
TODO

#### Class 文件结构的发展
TODO
