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
