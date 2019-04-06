### 将语法和程序的逻辑代码解耦

#### 从内嵌动作到监听器的演进
监听器和访问器机制能够将语法和程序代码解耦, 从而大有裨益; 这样的解耦封装起来, 避免了杂乱无章的分散在语法中; 如果语法没有内嵌动作, 就可以在多个程序中复用一个语法, 而无须为每个目标语法分析器重新编译一次; 受益于内嵌动作的机制, Antlr 能基于同一语法文件, 使用不同的编程语言生成语法分析器  
本节主要研究从包含内嵌动作的语法到完全与动作解耦的语法的演进过程; 下列语法用于读取属性文件, 这些文件的每行都是一个复制语句, 其中 <<...>> 是内嵌动作的概要
```
grammar PropertyFile;
file: {<<start file>>} prop+ {<<end file>>};
prop: ID '=' STRING '\n' {<<process property>>};
ID: [a-z]+;
STRING: '"' .*? '"';
```
这样的紧耦合使得语法被绑定到了特定的程序上; 更好的方案是, 从 Antlr 自动生成的语法分析器 PropertyFileParser 派生一个子类, 然后将内嵌动作转换为方法; 这样的重构可以使得语法中仅仅包含方法的调用, 之后可以实现不同功能的子类, 而无须修改原先的语法
```
grammar PropertyFile;
@members {
    void startFile() {}  // 空实现
    void endFile() {}
    void defineProperty(Token name, Token value) {}
}
file: {startFile();} prop+ {endFile();};
prop: ID '=' STRING '\n' {defineProperty($ID, $STRING)};
ID: [a-z]+;
STRING: '"' .*? '"';
```
但这份语法仍然存在缺陷: 受内嵌动作的限制, 它只能生成 Java 编写的语法分析器; 为了使语法可被重用并具有语言中立性, 我们需要完全避免内嵌动作的存在, 可以使用监听器和访问器来达到这个目的

#### 使用语法分析树监听器编写程序
构建应用逻辑和语法松耦合的语言类应用程序的关键在于, 令语法分析器建立一颗语法分析树, 然后在变量该树的过程中触发应用逻辑代码; 以下是一个识别属性文件的语法
```
grammar PropertyFile;

file: prop+;
prop: ID '=' STRING '\r'?'\n';

ID: [a-z]+;
STRING: '"' .*? '"';
```
Antlr 基于语法 PropertyFile 生成的监听器接口 `PropertyFileListener`; Antlr 的 `ParseTreeWalker` 在每次访问和离开节点的时候会分别触发对应规则子树的 enter 和 exit 方法; 因为语法 PropertyFile 中只有两条语法规则, 所以接口中有四个方法
```
public interface PropertyFileListener extends ParseTreeListener {
	void enterFile(PropertyFileParser.FileContext ctx);
	void exitFile(PropertyFileParser.FileContext ctx);
	void enterProp(PropertyFileParser.PropContext ctx);
	void exitProp(PropertyFileParser.PropContext ctx);
}
```
其中 `FileContext` 和 `PropContext` 类是每条语法规则对应的语法分析树节点的实现; Antlr 也提供了一个 `PropertyFileListener` 的默认实现, 因此只需要继承并实现所需方法即可
```
public class PropertyFileByListener extends PropertyFileBaseListener {

    private Map<String,String> props = new HashMap<>();

    public Map<String, String> getProps() {
        return props;
    }

    @Override
    public void exitProp(PropertyFileParser.PropContext ctx) {
        String id = ctx.ID().getText();
        String vlaue = ctx.STRING().toString();
        props.put(id,vlaue);
    }
}
```
监听器方法是在语法分析器完成解析之后才被触发的; 所以需要在完成语法解析之后, 再次遍历语法分析树, 并在这个过程中使用 `PropertyFileByListener` 监听相应的事件
```
// 新建一个监听器类
PropertyFileByListener loader = new PropertyFileByListener();
// 新建一个遍历器类, 并将监听器和语法树作为参数
new ParseTreeWalker().walk(loader, parser.file());
// 打印结果
System.out.println(loader.getProps());
```

#### 使用访问器编写程序
这里使用访问器代替监听器机制的详细步骤是: 令 Antlr 生成一个访问器接口, 实现该接口然后再编写一个对语法分析树调用 visit() 方法; 所以此方法也不需要与语法交互  
在命令行中使用 `-visitor` 选项, Antlr 会自动生成 `PropertyFileVisitor` 接口以及默认实现类; 接着继承默认实现类并实现所需方法即可
```
public class PropertyFileByVisitor extends PropertyFileBaseVisitor<Void> {

    private Map<String,String> props = new HashMap<>();

    public Map<String, String> getProps() {
        return props;
    }

    @Override
    public Void visitProp(PropertyFileParser.PropContext ctx) {
        String id = ctx.ID().getText();
        String value = ctx.STRING().toString();
        props.put(id, value);
        return null;  // 这里需要根据定义的泛型, 返回一个值
    }
}
```
访问器方法也是在语法分析器完成解析之后才被触发的; 所以需要在完成语法解析之后, 显示调用 `ParseTreeVisitor` 的 `visit()` 方法来遍历语法树, 在此过程中使用 `PropertyFileByVisitor` 访问相应的事件
```
// 新建一个访问器类
PropertyFileByVisitor visitor = new PropertyFileByVisitor();
// 调用 visit 访问语法树
visitor.visit(parser.file());
// 打印结果
System.out.println(visitor.getProps());
```

#### 标记备选分支以获取精确的事件方法
为了说明过粗的事件粒度带来的问题, 使用以下表达式语法生成的监听器构建一个简单的计算器程序
```
grammar Expr;
s: e;
e: e op=MULT e  // MULT 是 *
 | e op=ADD e   // ADD 是 +
 | INT
 ;

MULT: '*';
ADD: '+';
INT: [1-9][0-9]*;
WS : [ \t\n]+ -> skip;
```
规则 e 产生了一个相当鸡肋的监听器方法, 因为规则 e 的所有备选分支都会被遍历器触发为完全相同的 enterE() 和 exitE() 方法; 为此实现监听器时就不得不做判断: 使用 op 词法符号和 ctx 的方法来判qi断语法分析器匹配的子树 e 是哪一个备选分支
```
@Override
public void exitE(ExprParser.EContext ctx) {
    if (ctx.getChildCount() == 3) {
        int left = values.get(ctx.e(0));
        int right = values.get(ctx.e(1));
        if (ctx.op.getType() == ExprParser.MULT) {
            values.put(ctx, left * right);
        } else {
            values.put(ctx, left + right);
        }
    } else {
        // 一个 INT
        values.put(ctx, values.get(ctx.getChild(0)));
    }
}
```
其中 exitE() 引用的 MULT 字段是由 Antlr 在 ExprParser 中生成的; 为了获取更加精确的监听器事件, Antlr 允许使用 `#` 运算符为任意规则的最外层备选分支提供标签; 使用这种方法为 e 的备选分支增加标签
```
e: e MULT e  # Mult
 | e ADD e   # Add
 | INT
 ;
```
这样, Antlr 就会为 e 的每个备选分支都生成了一个单独的监听器方法, 就不再需要 op 这个词法符号标签了
```
public interface ExprListener extends ParseTreeListener {
    ...
	void enterAdd(ExprParser.AddContext ctx);
	void exitAdd(ExprParser.AddContext ctx);
	void enterMult(ExprParser.MultContext ctx);
	void exitMult(ExprParser.MultContext ctx);
	void enterInt(ExprParser.IntContext ctx);
	void exitInt(ExprParser.IntContext ctx);
}
```
需要注意的是, Antlr 也为不同的备选分支生成了特定的上下文对象 (EContext 的子类), 并以标签命名; 这样的上下文对象中的 getter 方法被限制为只能获取对应备选分支中的内容

#### 在事件方法中共享信息
有时候事件方法需要传递局部调用的结果或者其他信息; 但 Antlr 自动生成的监听器方法是不带自定义返回值和参数的, 生成的访问器也不带自定义参数; 本节会介绍三种在不改变事件方法签名的前提下传递数据的机制
- 使用访问器方法来返回值
- 使用类成员在事件方法之间共享数据
- 通过对语法分析树的节点进行标注来存储相关的数据

##### 使用访问器遍历语法分析树
为构建一个基于访问器的计算器程序, 最简单的方法是令 expr 规则中的事件方法返回子表达式的值; 为访问器方法增加返回值类型十分容易, 在继承 LExprBaseVisitor<T> 指定泛型参数即可
```
public class EvalVisitor extends LExprBaseVisitor<Integer> {

    @Override
    public Integer visitMult(LExprParser.MultContext ctx) {
        return visit(ctx.e(0)) * visit(ctx.e(1));
    }

    @Override
    public Integer visitAdd(LExprParser.AddContext ctx) {
        return visit(ctx.e(0)) + visit(ctx.e(1));
    }

    @Override
    public Integer visitInt(LExprParser.IntContext ctx) {
        return Integer.valueOf(ctx.INT().getText());
    }
}
```
EvalVisitor 从 Antlr 的 AbstractParseTreeVisitor 类继承了通用的 visit() 方法, 我们将它作为触发子树访问的捷径; 在 EvalVisitor 中没有 s 规则对应的访问器方法, 因为 LExprBaseVisitor 中其默认实现会调用预定义的 ParseTreeVisitor.visitChildren() 方法, 该方法返回访问最后一个子节点的返回值

##### 使用栈模拟返回值
Antlr 生成的监听器方法是没有返回值的的 (void 类型); 在语法分析树中, 为了向监听器方法的更高层调用者返回值, 我们可以将监听器的局部结果保存在一个成员变量中
```
public class EvalListener extends LExprBaseListener {

    private Stack<Integer> stack = new Stack<>();

    public Stack<Integer> getStack() {
        return stack;
    }

    @Override
    public void exitMult(LExprParser.MultContext ctx) {
        int right = stack.pop();
        int left = stack.pop();
        stack.push(right * left);
    }

    @Override
    public void exitAdd(LExprParser.AddContext ctx) {
        int right = stack.pop();
        int left = stack.pop();
        stack.push(right + left);
    }

    @Override
    public void exitInt(LExprParser.IntContext ctx) {
        stack.push(Integer.valueOf(ctx.INT().getText()));
    }
}
```
这种使用栈的方法不够优雅, 但是非常有效; 通过它可以保证事件方法在所有的监听器事件之间的执行顺序是正确的; 带返回值的访问器足够优雅, 但是需要我们手工触发对树节点的访问

##### 标注语法分析树
在上文中, 我们使用了临时存储在事件方法之间共享数据; 作为一个替代方案, 我们可以将这些数据直接存储在语法分析树里; 监听器和访问器机制都支持树的标注; 以下是带结果的 LExpr 语法分析树
```
                    s(7)
                     |
          --------- e(7) ---------
          |          |           |
          e(1)       +      --- e(6) ---
          |                 |     |    |
          1                 e(2)  *    e(3)
                            |          |
                            2          3
```
其中每个子表达式对应一个子树的根节点, 每个 e 节点拥有局部的计算结果; e 的备选分支对应的监听器将会在对应的语法分析树节点中保存一个结果; 在更高层的节点上, 后续的任何加法或者乘法事件触发的方法都可以通过查找对应的子节点中存储的值来获得子表达式的结果  
最简单的标注语法分析树节点的方法是使用 Map 来将任意值和节点一一对应起来; 出于这个目的 Antlr 提供了一个简单的名为 ParseTreeProperty 的辅助类; 如果想使用自己的 Map 代替此类, 请确保 Map 类是从 IdentityHashMap 派生而来, 因为这里使用同一性而非 equals() 来标注特定的节点
```
public class EvalProperty extends LExprBaseListener {

    private ParseTreeProperty<Integer> values = new ParseTreeProperty<>();

    public void setValue(ParseTree node, int value) {
        values.put(node, value);
    }

    public int getValue(ParseTree node) {
        return values.get(node);
    }

    @Override
    public void exitInt(LExprParser.IntContext ctx) {
        values.put(ctx, Integer.valueOf(ctx.INT().getText()));
    }

    @Override
    public void exitAdd(LExprParser.AddContext ctx) {
        int left = getValue(ctx.e(0));
        int right = getValue(ctx.e(1));
        values.put(ctx, left + right);
    }

    @Override
    public void exitMult(LExprParser.MultContext ctx) {
        int left = getValue(ctx.e(0));
        int right = getValue(ctx.e(1));
        values.put(ctx, left * right);
    }

    @Override
    public void exitS(LExprParser.SContext ctx) {
        setValue(ctx, getValue(ctx.e()));
    }
}
```

##### 不同的数据共享方法对比
为获取可复用的方法, 需要是其与用户自定义的动作分离; 这意味着将所有程序自身的逻辑代码放到语法之外的某种监听器或者访问器中; Antlr 生成的接口和默认实现类, 由于事件方法的签名是固定的, 无法由程序自行决定, 本节提供了以下三种共享数据的方案
- 原生的 Java 调用栈: 访问器返回一个用户指定类型的值, 不过如果访问器需要传递参数, 那就必须使用下面两种方案
- 基于栈的: 在上下文类中维护一个栈字段, 以与 Java 调用栈相同的方式, 模拟参数与返回值的入栈和出栈
- 标注: 在上下文类中维护一个 Map 字段, 用对应的值来标注节点

这三种方案把具体的逻辑封装在特定的对象内, 从而是其与语法本身完全解耦; 此外, 它们各有利弊, 需视情况选择使用
