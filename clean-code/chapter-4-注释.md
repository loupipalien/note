### 注释
什么也比不上放置良好的注释来得有用, 什么也不会比乱七八糟的注释更有本事搞乱一个模块, 什么也不会比陈旧的提供错误信息的注释更有破坏性; 真实只在一处地方有: 代码; 只有代码能忠实的告诉你它做的事情, 那是唯一真正准确的信息来源, 尽管有时也需要注释, 但更应该提高代码的表达力, 尽可能的减少注释

#### 注释不能美化代码
带有少量注释的整洁而有表达力的代码, 要比带有大量注释的零碎而复杂的代码像样的多; 与其花时间编写解释糟糕代码的注释, 不如花时间清洁那堆糟糕的代码

#### 用代码来阐述
有时代码本身不足以解释其行为, 但只要想上一点时间就能用代码解释大部分的意图; 很多时候简单到只需要创建一个描述与注释所养同一事物的函数即可
```
// Check to see if the employee is eligible for full benefits
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65))
```
使用代码描述
```
if (employee.isEligibleForBenefits())
```

#### 好注释
有些注释是必须的, 也是有利的; 不过要记住的是, 唯一真正好的注释是想办法不去写注释

##### 法律信息
有时公式代码规范要求编写与法律有关的注释; 例如, 版权及著作权声明就是必须和和有理由在每个源文件开头注释处放置的内容

##### 提供信息的注释
用注释来提供基本信息也有其用处; 例如, 以下注释解释了某个抽象方法的返回值
```
// Returns an instance of the Responder being tested.
protected abstract Responder responderInstance();
```
这类注释有时管用, 但更好的方式是尽量利用函数名称传达信息; 例如, 将上例中的函数重命名为 responderBeingTested, 注释就是多余的了

##### 对意图的解释
有时注释不仅提供了有关实现的有用信息, 而且还提供了某个决定后面的意图; 例如以下示例, 在对比两个对象时, 作者决定将自己的类放置在比其他东西更高的位置
```
public int compareTo(Object o) {
    if (o instanceof WikiPagePath) {
        WikiPagePath p = (WikiPagePath) o;
        String compressedName = StringUtil.join(names, "");
        String compressedArgumentName = StringUtil.join(p.names, "");
        return compressedName.compareTo(compressedArgumentName);
    }
    // we are greater because we are the right type
    return 1;
}
```

##### 阐释
有时, 注释把某些晦涩难明的参数会返回值的意义翻译为某种可读形式, 也是会有用的; 更好的方法是尽量让参数或返回值自身就足够清除, 但如果参数或返回值是某个标准库的一部分, 或者是你不能修改的代码, 帮助阐述其含义的代码就会有用, 但这也会冒着阐述性注释本身就不正确的风险; 所以在写注释之前, 考虑下是否有更好的方法, 然后再加倍小心的确认注释的正确性

##### 警示
有时, 用于警告其他程序员会出现某种后果的注释也是有用的; 例如在测试用例中, 可以利用附上恰当解释性字符串的 @Ignore 属性来关闭测试用例等

##### TODO 注释
有理由用 //TODO 形式在源代码中放置要做的工作列表; TODO 是一种程序员认为应该做的, 但由于某些原因目前还没做的工作; 无论 TODO 的目的如何, 它都不是在系统中留下糟糕代码的借口

##### 放大
注释可以用来放大吗某种看来不合理之物的重要性
```
// the trim is real important; It removes the starting space
// that could cause the item to be recognized as anther list
String listItemContent = match.group(3).trim();
new ListItemWidget(this, listItemContent, this.level + 1);
return buildList(text.substring(match.end()));
```

##### 公共 API 中的 Javadoc
没有什么比被良好描述的公共 API 更有用和令人满意的了, 如果在编写公共 API, 就该为它编写良好的 Javadoc

#### 坏注释
大多数注释都属此类; 通常坏注释都是糟糕的代码的支撑或借口, 或者对错误决策的修正, 基本上等于程序员自话自说

##### 喃喃自语
如果只是因为觉得应该或者过程需要就添加注释, 那就是无谓之举; 如果决定写注释, 就要花必要的时间确保写出最好的注释; 任何迫使读者查看其他模块的注释, 都没能与读者沟通好, 不值所费

##### 多余的注释
```
// Utility method that return when this.closed is true
// Throws an exception if the timeout is reached
public synchronized void waitForClose(final long timeoutMillis) throws Exception {
    if (!closed) {
        wait(timeoutMillis);
        if (!closed) {
          throws new Exception("MockResponseSender could not be closed.");
        }
    }
}
```
这段注释并不能比代码本身提供更多的信息, 它没有证明代码的意义, 也没有给出代码的意图或者逻辑; 读它并不比读代码容易

##### 误导性注释
程序员会写出不够精确的注释

##### 循规式注释
所谓每个函数都要有 Javadoc 或者每个变量都要有注释的规矩全然是愚蠢可笑的, 这类注释徒然会让代码变得散乱, 令人迷糊不解

##### 日志式注释
在模块开始时添加注释记录每次修改的日志, 在无代码控制工具时使用还算有道理, 但现在已无必要

##### 废话注释
对于显然之事喋喋不休, 毫无新意
```
/**
* Returns the day of month
* @return the day of month
*/
public int getDayOfMonth() {
  return dayOfMonth;
}
```

##### 可怕的废话
Javadoc 也可能是废话, 可能来源于某种提供文档的不当愿望的废话注释

##### 位置标记
有时, 程序员喜欢在源代码中标记某个特别的位置, 例如
```
// Actions /////////////////////////
```
如果标记栏不多就会显而易见, 但滥用标记栏, 就会沉没在背景噪音中被忽略掉

##### 括号后面的注释
对于含有深度嵌套结构的长函数可能有意义, 但是更好的做法是修改为更短下的函数, 就不用括号后的注释了

##### 归属与署名
```
// Added by micheal
```
源代码控制系统非常善于记住是谁在何时添加了什么, 没必要用这些注释来搞脏代码; 源代码控制系统是这类信息最好的归属地

##### 注释掉的代码
直接把代码注释掉是讨厌的做法; 会让读者不敢删除注释掉的代码, 但实际上这些被注释的代码根本没用; 如果需要找回代码, 使用源代码控制工具找回

##### HTML 注释
源代码注释中的 HTML 标记是一种讨厌物, HTML 注释的存在会导致代码变得异常难读; 如果注释将由某种工具 (如 Javadoc) 抽取出来呈现到网页, 那么该是工具而非程序员来负责给注释加上合适的 HTML

##### 非本地信息
注释请确保它描述了离它最近的代码, 别在本地注释的上下文环境中给出系统级的信息

##### 信息过多
别在注释中添加有趣的历史性话题或者无关的细节描述

##### 不明显的联系
注释及其描述的代码之间的联系应该显而易见, 至少让读者能看着注释和代码可以理解注释所谈何物; 注释的作用就是解释未能自行解释的代码, 如果注释本身还需要解释那就太遗憾了

##### 函数头
短函数不需要太多的注释; 为只做一件事的短函数取个好名字通常要比写函数头注释要好

##### 非公共代码中的 Javadoc
虽然 Javadoc 对于公共 API 非常有用, 但对于不打算作公共用途的代码就令人厌恶了; 因为其很少有用, 并且 Javadoc 注释额外的形式要求几乎等同于八股文章

##### 范例
TODO
