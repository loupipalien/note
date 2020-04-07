### 面向对象设计的原则

#### 单一职责原则 (Single Responsibility Principle, SRP)
就一个类而言, 应该仅有一个引起它变化的原因
- 如果一个类承担的职责过多, 就等于把这些职责耦合在一起, 一个职责的变化就可能会削弱或抑制这个类完成其他职责的能力, 这种耦合会导致脆弱的设计, 当变化发生时, 设计会遭受到意想不到的破坏
- 软件设计真正要做的是许多内容, 就是发现职责并把那些职责相互分离; 如果能够想到多于一个的冬季去改变类, 那么这个类就具有多于一个的职责

#### 开放封闭原则 (Open Closed Principle, OCP)
软件实体 (类, 模块, 函数等) 应该可以扩展, 但是不可以修改
- 对于扩展是开放的 (Open for extension)
- 对于更改是封闭的 (Closed for modification)

开放封闭原则是面向对象设计的核心所在; 遵循这个原则可以带来面向对象技术所声称的巨大好处, 也就是可维护, 可扩展, 可复用, 灵活性好; 开发人员应该仅对程序中呈现出频繁变化的那些部分做出抽象, 然而对于应用程序中的每个部分都刻意的进行抽象同样不是一个好主意, 拒绝不成熟的抽象和抽象本身一样重要

#### 依赖倒置原则 (Dependency Inversion Principle, DIP)
高层模块不应该依赖底层模块, 两个都应该依赖于抽象; 抽象不应该依赖细节, 细节应该依赖于抽象  
具体来说就是面向接口编程, 而不是面向具体实现编程

#### 里氏替换原则 (Liskov Substitution Principle, LSP)
子类型必须能够替换掉它们的父类型  
一个软件实体如果使用的是一个父类的话, 那么一定适用于其子类, 而且它察觉不出父类对象和子类对象的区别; 也就是说, 把父类都替换为它的子类, 程序的行为没有变化

#### 接口分离原则 (Interface Segregation Principle, ISP)

#### 最少知识原则 (Least Knowledge Principle, LKP)
也叫 `迪米特法则`; 如果两个类不必彼此直接通信, 那么这两个类就不应当发生直接的相互作用; 如果其中一个类需要调用另一个类的某一个方法的话, 可以通过第三者转发这个调用   
其根本思想是强调了类之间的松耦合, 类之间的耦合越弱, 越有利于复用;一个处在弱耦合的类被修改, 不会对有关系的类造成波及, 也就是说信息的隐藏促进了软件的复用

>**参考**
- [大话设计模式](https://book.douban.com/subject/2334288/)  
- [单一职责原则](http://www.cnblogs.com/gaochundong/p/single_responsibility_principle.html)  
- [开放封闭原则](http://www.cnblogs.com/gaochundong/p/open_closed_principle.html)  
- [依赖倒置原则](http://www.cnblogs.com/gaochundong/p/dependency_inversion_principle.html)  
- [里氏替换原则](http://www.cnblogs.com/gaochundong/p/liskov_substitution_principle.html)
- [接口分离原则](http://www.cnblogs.com/gaochundong/p/interface_segregation_principle.html)  
- [最少知识原则](http://www.cnblogs.com/gaochundong/p/least_knowledge_principle.html)
