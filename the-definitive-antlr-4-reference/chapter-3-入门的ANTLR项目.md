### 入门的 ANTLR 项目
目标是使用语法分析器将 Java 的 short 数组转换为字符串; 即将 short 值当作 Unicode 字符, 从而将 `static short[] data = {1, 2, 3}` 转换为等价的字符串形式 `static String data = "\u0001\u0002\u0003"`; 像 \u0001 这样的 Unicode 字符标记使用四个十六进制数字来表示一个 16 位的字符, 实际上这样的字符就是一个 short 的值

#### Antlr 工具, 运行库以及自动生成的代码
完成此项工作的步骤大概是: 定义一个可以描述目标语言的语法, 然后对语法运行 Antlr, 再将生成的代码与 jar 包中的运行库一起编译, 最后将编译好的代码和运行库放在一起运行  
构建一个语言类应用程序的第一步是创建一个能够描述这种语言的语法
```
# ArrayInit.g4
/**
 * 语法文件通常以 grammar 关键字开头
 * 这是一个名为 AarryInit 的语法, 它必须和文件名 ArrayInit.g4 相匹配
 */
grammar ArrayInit;

// 一条名为 init 的规则, 它匹配一对花括号中的, 逗号分隔的 value
init : '{' value (',' value)* '}' ;  // 至少匹配一个 value

// 一个 value 可以是嵌套的花括号结构, 也可以是一个简单的整数, 即 INT 词法符号
value : init
      | INT
      ;

// 语法分析器的规则必须以小写字母开头, 词法分析器的规则必须用大写字母开头
INT : [0-9]+ ;  // 定义词法符号 INT, 它由一个或多个数字组成
WS : [ \t\r\n]+ -> skip ;  // 定义词法规则 "空白符号", 丢弃之
```
编写完成后对语法运行 Antlr, 生成了一些文件
- ArrayInitParser.java
该文件包含一个语法分析器类的定义, 这个语法分析器专门用来识别定义的 "数组语言" 的语法 ArrayInit, 该类中每条惠泽都有对应的方法, 除此之外还有一些辅助代码
- ArrayInitLexer.java
Antlr 能够自动识别出语法中的文法规则和词法规则; 这个文件包含的是词法分析器的定义, 它是由 Antlr 通过分析词法规则 INT 和 WS, 以及语法中的字面值 '{', ',', '}' 生成的; 词法分析器的作用是将输入的字符序列分解成词汇符号
- ArrayInit.tokens
Antlr 会给定义的词法符号指定一个数字形式的类型, 然后将它们的对应关系存储于该文件中
- ArrayInitListener.java, ArrayInitBaseListener.java
默认情况下, Antlr 生成的语法分析器能够将输入文本转换为一颗语法分析树; 在遍历语法分析树时, 遍历器能够触发一系列的 "事件" (回调), 并通知我们提供的监听器对象; ArrayInitListener 接口给出了这些回调方法的定义, 可以实现它来完成自定义的功能; ArrayInitBaseListener 是该接口的默认实现类, 为其中的每个方法提供了一个空实现
- 其他文件
ArrayInit.interp, ArrayInitLexer.interp, ArrayInitLexer.tokens

#### 测试生成的语法分析器
编译生成的 Java 代码, 并使用 TestRig 进行测试...

#### 将生成的语法分析器与 Java 程序集成
在语法准备就绪后, 就可以将 Antlr 自动生成的代码与自己的程序集成; 以下示例展示调用语法解析器并打印出 `TestRig -tree` 类似的语法分析树
```
import org.antlr.v4.runtime.ANTLRInputStream;
import org.antlr.v4.runtime.CommonTokenStream;
import org.antlr.v4.runtime.tree.ParseTree;

public class Test {

    public static void main(String[] args) throws Exception {

        // 新建一个 CharStream, 从标准输入读取数据
        ANTLRInputStream input = new ANTLRInputStream(System.in);

        // 新建一个词法分析器, 处理输入的 CharStream
        ArrayInitLexer lexer = new ArrayInitLexer(input);

        // 新建一个词法符号的缓冲区, 用于存储词法分析器将生成的词法符号
        CommonTokenStream tokens = new CommonTokenStream(lexer);

        // 新建一个语法分析器, 处理词法符号缓冲区中的内容
        ArrayInitParser parser = new ArrayInitParser(tokens);

        // 针对 init 规则, 开始语法分析
        ParseTree tree = parser.init();
        // 使用 LISP 风格打印生成的树
        System.out.println(tree.toStringTree(parser));
    }
}
```

#### 构建一个语言类应用程序
为了完成我们的转换工作, 程序必须能够从语法分析树中提取数据; 最简单的方案是使用 Antlr 内置的语法分析树遍历器进行深度优先遍历, 然后在它触发一系列回调函数中进行准换操作; 使用监听器只需要继承 ArrayInitBaseListener, 然后覆盖其中必要方法即可; 使用监听器的好处在于不需要自己编写任何遍历语法分析树的代码  
一个进行翻译工作的项目意味着要处理这样的问题: 如何将输入的词法符号或词组翻译成输出字符串, 这里我们定义出转换逻辑
- 将 `{` 翻译为 `"`
- 将 `}` 翻译为 `"`
- 将每个整数翻译为四位的十六进制形式, 然后加上前缀 `\u`

以下是遵循翻译逻辑的监听器类的实现
```
public class ShortToUnicodeString extends ArrayInitBaseListener {

    /**
     * 将 { 翻译为 "
     * @param ctx
     */
    @Override
    public void enterInit(ArrayInitParser.InitContext ctx) {
        System.out.println('"');
    }

    /**
     * 将 } 翻译为 "
     * @param ctx
     */
    @Override
    public void exitInit(ArrayInitParser.InitContext ctx) {
        System.out.println('"');
    }

    /**
     * 将每个整数翻译为四位的十六进制形式, 然后加上前缀 \u
     * @param ctx
     */
    @Override
    public void enterValue(ArrayInitParser.ValueContext ctx) {
        // 假设不存在嵌套结构
        int value = Integer.valueOf(ctx.INT().getText());
        System.out.printf("\\u%04x", value);
    }
}
```
这里只需覆盖需要的方法即可; 上述代码中的 ctx.INT() 为上下文对象中获得的 INT 词法符号, 再取该词法符号对应的整数值转换; 最后我们将 Test 程序扩展为一个翻译程序
```
import org.antlr.v4.runtime.ANTLRInputStream;
import org.antlr.v4.runtime.CommonTokenStream;
import org.antlr.v4.runtime.tree.ParseTree;
import org.antlr.v4.runtime.tree.ParseTreeWalker;

public class Translate {

    public static void main(String[] args) throws Exception {

        // 新建一个 CharStream, 从标准输入读取数据
        ANTLRInputStream input = new ANTLRInputStream(System.in);

        // 新建一个词法分析器, 处理输入的 CharStream
        ArrayInitLexer lexer = new ArrayInitLexer(input);

        // 新建一个词法符号的缓冲区, 用于存储词法分析器将生成的词法符号
        CommonTokenStream tokens = new CommonTokenStream(lexer);

        // 新建一个语法分析器, 处理词法符号缓冲区中的内容
        ArrayInitParser parser = new ArrayInitParser(tokens);

        // 针对 init 规则, 开始语法分析
        ParseTree tree = parser.init();

        // 新建一个通用的, 能够触发回调函数的语法分析器遍历树
        ParseTreeWalker walker = new ParseTreeWalker();
        // 遍历语法分析过程中生成的语法分析树, 触发回调
        walker.walk(new  ShortToUnicodeString(), tree);
        // 翻译完成后换行
        System.out.println();
    }
}

```
