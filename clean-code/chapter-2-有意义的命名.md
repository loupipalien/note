### 有意义的命名

#### 介绍
给变量, 函数, 参数, 类和封包命名等等

#### 名副其实
选个好名字要花时间, 但省下来的时间比花掉的多; 注意命名, 一旦发现有好名字就替换掉旧的, 这么做会使读代码的人会很开心; 变量, 函数或类的名称应该已经答复了所有的大问题, 它该告诉你, 它为什么会存在, 它做什么事, 应该怎么用; 如果名需要注释来补充, 那就不算名副其实了  
选择体现本意的名称能让人更容易理解和修改代码; 下列代码的目的何在?
```
public List<int[]> getThem() {
    List<int[]> list1 = new ArrayList<int[]>();
    for (int[] x : theList) {
      if (x[0] == 4) {
        list1.add(x);
      }
    }
    return list1;
}
```
为什么难以说明以上代码要做什么事, 问题不在于代码的间接度, 而是在于代码的 **模糊度**: 即上下文在代码中未被明确体现的程度; 上述代码要求我们了解类似以下问题的答案:
- theList 中是什么类型的东西?
- theList 零下标条目的意义是什么?
- 值 4 的意义是什么?
- 怎么使用返回的列表?

#### 避免误导
程序员必须避免留下掩藏代码本意的错误线索, 应当避免使用与本意相悖的词; 例如: hp, aix, sco 都不该用做变量名, 因为它们都是 Unix 或类 Unix 平台的专有名称  
不要使用小写字母 l 和大写字母 O 做变量名, 尤其在组合使用的时候; 它们看起来完全像是常量 1 和 0

#### 做有意义的区分
以数字系列命名 (a1, a2, ... aN) 是依义命名的对立面; 这样的名称纯属误导 --- 完全没有提供正确的信息, 没有提供导向作者意图的线索; 例如
```
public static void copyChars(char a1[], char a2[]) {
    for (int i = 0; i < a.length; i++) {
        a2[i] = a1[i];
    }
}
```
如果参数名改为 source 和 destination, 这个函数就会像样很多  
废话是另一中没意义的区分; 假设有一个 Product 类, 如果还有 ProductInfo 或 ProductData 类, 虽然它们的名称不同, 意思却无区别; Info 和 Data 就像是 a, an, the 一样, 都是没意义的废话  
废话都是冗余; Variable 一词永远不应该出现在变量名中, Table 一词永远不应当出现在表名中,
NameString 会比 Name 好么? 难道 Name 会是一个浮点数不成? 设想有一个 Customer 类, 还有一个 CustomerObject 的类, 区别又在那里呢?  
如果缺少明确约定, 变量 moneyAmount 就与 money 没区别, customerInfo 与 customer 没区别, accountData 与 account 没区别, theMessage 与 message 没区别; 要区分名称, 就要以读者能鉴别不同之处的方式来区分

#### 使用都得出来的名称
人类擅长于记忆和使用单词, 大脑相当一部分是用来容纳和处理单词的, 如果不善加利用, 实在是种羞耻; **编程本身就是一种社交活动**, 变量名应该读的出来, 这样才方便交流讨论; 以下示例
```
// A
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
}
// B
class Customer {
    private Date generationTimestamp;
    private Date modificationTimestamp;
    private final String recordId = "102";
}
```

#### 使用可搜索的名称
单字母名称和数字常量有个问题, 就是很难在一大篇文字中找出来; 例如找 MAX_CLASSES_PER_STUDENT 很容易, 但是想找到数字 7 就比较麻烦了, 因为它可能是某些文件名或者其他常量定义的一部分, 出现在因不同意图而采用的各种表达式中; 如果常量是个长数字, 又被人错改过, 就很容易逃过搜索从而造成错误  
由此可见, 长名称胜于短名称, 搜索的到的名称胜于自造编码代写就的名称; 窃以为单字母名称仅用于短方法的本地变量; **名称长短应与作用域大小相对应**

#### 避免使用编码
编码已经太多, 无谓再自找麻烦, 把类型或作用域编进名称里面, 徒然增加了解码的负担

##### 匈牙利语标记法
以往 Fortan 语言要求首字母体现出类型, 导致了编码的产生; Basic 早期版本只允许使用一个字母再加上以为数字; 匈牙利语标记法 (Hungarian Notation, HN) 将这种态势愈演愈烈  
Java 程序员不需要类型编码, 对象是强类型的, 代码编辑环境已经先进到在编译开始前就侦测到类型错误的程度! 所以, 如今 HN 和其他类型编码都是多余的; 它们增加了修改变量, 函数或类的名称或类型的难度, 增加了阅读代码的难度, 制造了让编码系统误导读者的可能性

##### 成员前缀
也不必使用 m_ 前缀来标明成员变量, 应当把类和函数做的足够小, 消除对成员前缀的需要; 最好使用可以高亮或用颜色标出成员的编译环境
```
// A
public class Part {
    private String m_dec;
    void setName(String name) {
        m_dec = name;
    }
}
// B
public class Part {
    String description;
    void setDescription(String description) {
        this.description = description;
    }
}
```

##### 接口和实现
当在做一个创建形状用的抽象工厂 (Abstract Factory); 该工厂是个接口, 要用具体类来实现; 如何命名接口和实现类?  
IShapeFactory : ShapeFactory -> ShapeFactory : ShapeFactoryImpl, 选择在实现类的接口加名称编码也许来得更好些

#### 避免思维映射
不应当让读者在脑中把你的名称翻译为他们熟知的名称, 这种问题常常出现在选择使用问题领域术语还是解决方案领域术语时; 专业的程序员应当了解 **明确是王道**, 专业的程序员应当善用其能, 编写其他人能理解的代码

#### 类名
类名和对象名应该是名词或名词短语, 类名不应当是动词

#### 方法名
方法名应当是动词或者动词短语; 属性访问器, 修改器和断言应当根据其值命名, 并依据 Javabean 标准加上 get, set 和 is 前缀  
重载构造器时, 使用描述了参数的静态工厂方法名: Complex fulcrumPoint = Complex.FromRealNumber(23.0) 通常好于 Complex fulcrumPoint = new Complex(23.0), 可以考虑将相应的构造器设置为 private, 强制使用这种命名手段

#### 别扮可爱
如果名称太耍宝, 那就只有同作者一般具有幽默感的人才能记得住, 而且还是在她们记得那个笑话的时候才行; 扮可爱的做法在代码中经常体现为使用俗语或俚语: 别用 eatMyShorts() 这类与文化紧密相关的笑话来表示 abort()

#### 每个概念对应一个词
给每个抽象的概念选一个词, 并且一以贯之; 例如, 使用 fetch, retrieve, get 来给在多个类中的同种方法命名, 很难记得住哪个类中的哪个方法是用来做什么的, 就得耗费大把的时间浏览各个文件头及前面的代码; 再调用方法时, 编辑器通常不会给出函数名和参数列表的编写的注释, 如果参数名来自函数声明, 那必是极好的; 函数名称应当独一无二, 而且要保持一致, 这样才不用借助多余的浏览就能找到正确的方法; 对于类名也是, 同类型的类使用相同的前缀或后缀, 一贯以之的命名法是天降福音

#### 别用双关语
避免将同一单词用于不同目的, 同一术语用于不同概念, 基本上就是双关语了;  如果遵循 "一词一义" 规则, 可能在好多类中都会有 add 方法, 只要这些 add 方法的参数列表和返回值在语义上等价就一切顺利; 代码作者应尽力写出易于理解的代码, 而不必殚精竭虑的研究; 推崇哪种大众化的作者尽责写清楚的平装书模式, 尽量避免哪种学者挖地三尺才能明白个中意义的学院派模式

#### 使用解决方案领域名称
只有程序员才会读你的代码, 所以尽管使用那些计算机科学术语, 算法名, 模式名, 数据术语吧; 依据问题所涉及领域命名可能不是聪明的做法, 因为不该让你的协作者总是跑去问客户每个名称的含义, 他们早该通过另一名称了解这个概念了; 对于熟悉访问者模式的程序员来说, 名称 AccountVisitor 富有含义, 程序员要做太多技术性工作了, 给这些事物取个技术性的名称通常是最靠谱的做法

#### 使用源自所涉问题领域的名称
如果不能用程序员熟悉的术语来给手头的工作命名, 就采用从所涉问题领域而来的名称; 至少负责维护代码的程序员就能去请教领域专家了; 程序员的工作之一就是分离解决方案领域和问题领域的概念, 与所涉问题领域更为贴近的代码, 应当采用源自问题领域的名称

#### 添加有意义的语境
很少有名称是能自我说明的 --- 多数都不能; 反之, 需要用有良好命名的类, 函数, 名称空间来放置名称, 给读者提供语境; 如果没这么做, 给名称添加前缀就是最后一招了  
以下代码中的变量是否需要更有意义的语境呢? 函数名仅给出了部分语境, 算法提供了一部分; 便览函数后, 你就会知道 number, verb, pluralModifier 这三个变量是测估信息的一部分, 不幸的是这语境得靠读者自己推断出来; 第一眼看到这个方法时, 这些变量的含义完全不清楚
```
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluralModifier;
    if (count == 0) {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    } else if (count == 1) {
        number = "1";
        verb = "is";
        pluralModifier = "";
    } else {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }
    String guessMessage = String.format("There %s %s %s%s", verb, number, candidate, pluralModifier);
    print(guessMessage);
}
```
上述函数有点长, 变量的使用贯穿始终; 要分解这个函数, 需要创建一个名为 GuessStatisticsMessage 的类, 把这三个变量做成该类的成员字段; 这样它们就在定义上变作了 GuessStatisticsMessage 的一部分; 语境的增强也让算法能够通过分解为更小的函数而变得更为干净利落
```
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;

    public String make(char cnadidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format("There %s %s %s%s", verb, number, candidate, pluralModifier);
    }

    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count)
        }
    }

    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }

    private void thereIsOneLetter() {
        number = "1";
        verb = "is";
        pluralModifier = "";
    }

    private void thereAreNoLetters() {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
```

#### 不要添加没用的语境
只要短名称足够清楚, 就要比长名称好; 不要给名称添加不必要的语境

#### 最后的话
取好名字最难的地方在于需要良好的描述技巧和共有的文化背景; 与其说这是一种技术, 商业或管理问题, 还不如说是一种教学问题
