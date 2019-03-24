### 初识 ANTLR

#### 安装
Antlr 是用 Java 编写的, 因此首先需要安装 Java (JDK 6+), 哪怕你的目标是使用 Antlr 来生成其他语言 (如 C# 或者 C++ 等) 的解析器  
安装 Antlr 仅仅需要下载最新的 jar 包 (如 antlr-4.7.2-complete.jar), 然后放在合适的目录 (如 centos 机器的 /app 目录); 该 jar 包含了运行 Antlr 的工具和编译, 执行 Antlr 产生的识别程序所依赖的全部运行库  
后续操作都是使用命令行, 为了方便使用 Antlr, 可将其 jar 包声明在 CLASSPATH 中
```
# 其中的点非常重要, 它表示当前目录, 这可以使 Java 虚拟机加载当前目录中的 class 文件
$ export CLASSPATH=.:/app/antlr-4.7.2-complete.jar:$CLASSPATH
```
可以通过两种方法来确定 Antr 是否安装正确
```
# 使用 java -jar 直接运行 Antlr 的 jar 包启动 org.antlr.v4.Tool
$ java -jar /app/antlr-4.7.2-complete.jar
# 直接调用 org.antlr.v4.Tool
java -cp /app/antlr-4.7.2-complete.jar org.antlr.v4.Tool
```
后续输入这些命令会十分频繁, 可以将其定义为 alias 来简化输入
```
$ alias antlr4='java -jar /app/antlr-4.7.2-complete.jar'
```

#### 运行 Antlr 并测试识别程序
下面是一个简单的识别类似 hello world 的词组的语法文件 Hello.g4 (最好将其放在一个空文件夹中)
```
grammar Hello;                // 定义一个名为 Hello 的语法
r : 'hello' ID ;              // 匹配一个关键字 hello 和一个紧随其后的标识符
ID : [a-z]+ ;                 // 匹配小写字母组成的标识符
WS : [ \t\r\n]+ -> skip ;     // 忽略空格, Tab, 换行以及 \r (Windows)
```
接下来使用 Antlr 命令编译, 使用之前定义的 antlr4 命令生成语法分析器和词法分析器
```
$ antlr4 Hello.g4
```
Antlr 在运行库中提供了一个名为 TestRig 的调试工具; 它可以详细列出一个语言类应用程序在匹配输入文本过程中的信息, 这些输入文本可以来自文件或者标准输入, 其使用 Java 的反射机制来调用编译后的识别程序; 为了方便起见也使用一个 alias 来简化输入
```
# org.antlr.v4.runtime.misc.TestRig 已废弃
# 如果之前在 CLASSPATH 中没有声明 antlr-4.7.2-complete.jar, 则h注意要在这里指定
alias grun='java org.antlr.v4.gui.TestRig'
```
该测试组件接收一个语法名和一个起始规则名作为参数, 此外它还可以接收众多参数指定输出内容; 如果希望显示识别过程中生成的词法符号, 则可以使用以下命令
```
$ grun Hello r -tokens    # 使用 Hello 语法和 r 规则启动 TestRig
hello parrt               # 键入要识别的语句
EOF                       # 在 Unix 上是 Ctrl + D, Windows 上是 Ctrl + Z 来输出文件结束符
# 以下三行是输出
[@0,0:4='hello',<'hello'>,1:0]
[@1,6:10='parrt',<ID>,1:6]
[@2,12:11='<EOF>',<EOF>,2:0]
```
每行输出代表一个词法符号, 其中包含了该词法符号的全部信息; `[@1,6:10='parrt',<ID>,1:6]` 表示: 这个词法符号位于第二个位置 (从 0 开始计数), 由输入文本的第 6 个和第 10 个位置之间的字符组成 (同样从 0 开始计数), 包含的文本内容是 parrt, 词法符号类型是 2 (即 ID), 位于文本第一行的第 6 个位置处 (从 0 开始技术, tab 符号被看作一个字符)  
还可以使用其他命令来打印语法分析树 (`grun Hello r -tree`) 或者可视化语法分析树 (`grun Hello r -gui`, 注意需要支持图形化环境, 见 [issue](https://github.com/antlr/antlr4/issues/844)); grun 还支持以下参数

| 参数 | 说明 |
| :--- | :--- |
| -tokens | 打印出词法符号流 |
| -tree | 以 LISP 格式打印出语法树 |
| -gui | 在对话框中以可视化的方式显示语法分析树 |
| -ps file.ps | 以 PostScript 格式生成可视化语法分析树, 然后将其存储于 file.ps |
| -encoding encodingname | 若当前的区域设定无法正确读取输入, 使用这个选项指定测试组件输入文件的编码 |
| -trace | 打印规则的名字以及进入和离开规则时的词法符号 |
| -diagnostics |开启解析过程中的挑食信息输出 |
| -SSL | 使用另外一个更快但是功能稍弱的解析策略 |
