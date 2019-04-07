### 属性和动作
之前学习的程序逻辑代码都是与语法分析树遍历器分离的, 这意味着代码总是在语法分析完成之后执行; 但一些应用程序需要在语法分析的过程中执行自身的逻辑代码, 为了达到这个目的, 我们采用了一种将代码片段直接注入在 Antlr 生成的代码中的方法 --- 动作  
通常我们应当避免将语法和应用程序的逻辑代码绑定在一起; 不包含动作的语法更易阅读, 也不会绑定到特定目标语言和程序上; 尽管如此, 内嵌动作仍然有用武之地, 原因如下
- 简便: 有时使用少量的动作, 避免创建一个监听器或者访问器, 会使得事情变得更加简单
- 效率: 在资源紧张的程序中, 可能步响把宝贵的时间和内存浪费在建立语法分析树上
- 带判定的语法分析过程:  在某些罕见的情况下, 我们必须依赖从之前的输入流中获取的数据才能正常的进行语法分析过程; 例如一些语法需要建立一个符号表, 以便在未来根据情况差异化识别输入的文本

动作就是使用目标语言编写的, 放置在 `{...}` 中的任意代码块, 我们可以在动作中编写任意代码, 只要它们是合法的目标语言语句

##### 使用动作的语法编写一个计算器
我们的目标是在不使用访问器, 甚至不建立语法分析树的前提下, 重新编写一个与之前编写的功能相同的计算器  
在本节中将会了解到以下技能: 将生成的语法分析器放入包中, 定义语法分析器的字段和方法, 在备选分支中插入动作, 标记语法元素以便在动作中使用, 以及定义规则的返回值

##### 在语法规则之外使用动作
在语法规则之外, 我们希望将两种东西注入自动生成的语法分析器和词法分析器: `package/import` 语句以及类似字段和方法的类成员  
我们可以在语法中使用 `@header {...}` 来指定一段 header 动作代码, 使用 `@members {...}` 向生成的代码中注入字段或者方法; 这些具名的动作会同时应用于语法分析器和词法分析器 (Antlr 选项 -package 允许我们直接设定包名, 而无需使用 header 动作); 如果需要限制一段动作代码值出现在语法分析器或者词法分析器中, 则可以使用 `@parser:name` 或者 `@lexer:name`

##### 在规则中嵌入动作
在本节中将会学习在语法中嵌入动作, 这些动作而可以生成输出, 更新数据结构, 或者设置规则的返回值; 还会看到 Antlr 是如何将规则的参数, 返回值, 以及规则调用的其他属性包装成一个 ParserRuleContext 子类的实例
###### 基础知识
stat 规则用于识别表达式, 变量赋值语句和空行; 因为在空行时什么都不做, 因此只需要两个动作
```
stat:   e NEWLINE           {System.out.println($e.v);}
    |   ID '=' e NEWLINE    {memory.put($ID.text, $e.v);}
    |   NEWLINE                   
    ;
```
动作被执行的时机是它前面的语法元素之后, 它后面的语法元素之前; Antlr 中 `$x.y` 形式的调用, x 是词法符号引用和语法规则引用, y 是 x 的属性 (如果 Antlr 无法识别 y 属性, 则不会转换该属性); 这里的 `$e.v` 和 `$ID.text` 分别指调用语法规则 e 的返回值 (稍后可以看到为什么是 v) 和 ID 词法符号匹配到的文本; 以下是语法规则 e
```
e returns [int v]
    : a=e op=('*'|'/') b=e  {$v = eval($a.v, $op.type, $b.v);}
    | a=e op=('+'|'-') b=e  {$v = eval($a.v, $op.type, $b.v);}  
    | INT                   {$v = $INT.int;}    
    | ID
      {
      String id = $ID.text;
      $v = memory.containsKey(id) ? memory.get(id) : 0;
      }
    | '(' e ')'             {$v = $e.v;}       
    ;
```
这里 e 指定了一个整数类型的返回值 v, 这就是之前 `$e.v` 的原因

###### 将一切打包成一个规则上下文对象
在之前我们已经了解到, Antlr 通过语法规则上下文对象 (rule context object) 来实现语法分析树的节点; 每次规则调用都会新建一个语法规则上下文对象, 它存储了相应语法规则在输入流的特定位置上进行识别工作的所有重要信息; EContext 的部分代码如下所示
```
public static class EContext extends ParserRuleContext {
    public int v;           // 语法规则 e 的返回值, 源于 "ruturn [int v]"
    public EContext a;      // 语法规则引用 e 上的标记 a
    public Token INT;       // 第四个备选分支引用 INT
    public Token ID;        // 第三个备选分支引用 ID
    public EContext e;      // e 的调用过程对应的上下文对象的引用
    public Token op;        // 类似 ('*'|'/') 的运算符子规则的标记
    public EContext b;      // 语法规则引用 e 上的标记 b
    ...
}
```
标记总会称为语法规则上下文对象的成员, 但是 Antlr 并不总为类似 ID, INT 和 e 的备选分支元素生成字段; Antlr 只有它们被语法中的动作引用时才为它们生成字段, 因为 Antlr 会尽可能的减少上下文对象中字段的数量
###### 计算返回值
e 中的所有动作都通过赋值语句 "$v = ...;" 来设置返回值, 该语句虽然设置了返回值, 但是并不会导致对应的规则函数返回 (不要在动作中使用 return 语句, 这会使语法分析器崩溃)

###### 编写一个交互式的计算器
为使得这个过程变成交互式的, 输入文本必须一行一行的传递 (如果需要分为多行的表达式, 处理过程更为复杂一些)
```
public static void main(String[] args) throws Exception {
   InputStream is = args.length > 0 ? new FileInputStream(args[0]) : System.in;
   BufferedReader br = new BufferedReader(new InputStreamReader(is));

   // 共享同一个语法分析器的实例
   ExprParser parser = new ExprParser(null);
   // 不需要建立语法分析树
   parser.setBuildParseTree(false);

   // 获取第一个表达式
   String expr = br.readLine();
   // 跟踪输入的表达式的行号
   int line = 1;
   // 当多于一个表达式时
   while ( expr != null ) {
       // 为每行建立一个词法分析器和词法符号流
       CharStream input = CharStreams.fromString(expr+"\n");
       ExprLexer lexer = new ExprLexer(input);
       // 通知词法分析器输入的位置
       lexer.setLine(line);
       lexer.setCharPositionInLine(0);
       CommonTokenStream tokens = new CommonTokenStream(lexer);
       // 使用新的词法符号流通知语法分析器
       parser.setInputStream(tokens);
       // 开始语法分析过程
       parser.stat();
       // 读取下一行
       expr = br.readLine();
       line++;
   }
}
```

#### 访问词法符号的规则和属性
TODO

#### 识别关键字不固定的语言
Java 5 中新增了关键字 enum, 比编译器可以更具 "-version" 选项动态的开启和关闭它; 也许更常见的引用是处理拥有巨量关键字集合的语言; 我们可以令词法分析器分别匹配所有的关键字, 也可以编写一条 ID 规则作为分发器, 然后在一个关键字列表中查找该规则匹配的标识符; 如果词法分析器发现该标识符是一个关键字, 我们可以用它的词法符号类型从原先通用的 ID 类型改成相应的关键字类型  
在实现 ID 规则和关键字查找机制之前, 先来看包含关键字引用的语句规则
```
stat:   BEGIN stat* END
    |   IF expr THEN stat
    |   WHILE expr stat
    |   ID '=' expr ';'
	;

expr:   INT | CHAR ;
```
虽然 Antlr 会隐式的为每个关键字 (BEGIN, END 等) 定义一个词法符号类型, 但是会收到警告需显示定义; 为关闭这个警告定义如下
```
// 显示定义关键字的词法符号类型, 避免隐式定义产生的警告
tokens { BEGIN, END, IF, THEN, WHILE }
```
接下来使用一个 Map 存放关键字到其整数词法符号的类型的映射作为关键字表
```
// 只在词法分析器中放置这个成员
@lexer::members {
Map<String,Integer> keywords = new HashMap<String,Integer>() {{
    put("begin", KeywordsParser.BEGIN);
    put("end",   KeywordsParser.END);
    put("if",    KeywordsParser.IF);
    put("then",  KeywordsParser.THEN);
    put("while", KeywordsParser.WHILE);
}};
```
最后在 ID 词法规则匹配时将词法符号类型设置为恰当的值
```
ID: [a-zA-Z]+
    {
        if ( keywords.containsKey(getText()) ) {
            // 重置词法符号类型
            setType(keywords.get(getText())); // reset token type
        }
    };
```
