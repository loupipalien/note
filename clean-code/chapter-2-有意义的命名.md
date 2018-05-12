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
