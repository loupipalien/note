### 访问者模式 (Visitor Pattern)
表示一个作用于某对象结构中的各元素的操作, 可以在不改变各元素的类的前提下定义作用于这些元素的新操作; 访问者模式属于行为型模式

#### 模式角色
- AbstractVisitor: 抽象访问者类
为对象结构中每一个具体元素类 ConcreteElement 声明一个访问操作
- ConcreteVisitor: 具体访问者类
实现有抽象访问类声明的操作, 每个操作用于访问对象结构中的一种类型元素
- AbstractElement: 抽象元素类
定义一个 accept() 方法, 该方法通常以抽象访问者实例作为参数
- ConcreteElement: 具体元素类
实现 accept() 方法, 在 accept() 方法中调用访问者的访问方法以完成对一个元素的操作
- ObjectStructure: 对象结构类
元素的集合, 用于存放元素对象, 并且提供了遍历其内部元素的方法

#### 代码示例
```
// 抽象访问者类
public abstract class AbstractVisitor {

    public abstract void visitConcreteElementA(ConcreteElementA elementA);

    public abstract void visitConcreteElementB(ConcreteElementB elementB);
}

// 具体访问者类
public class ConcreteVisitor extends AbstractVisitor {

    @Override
    public void visitConcreteElementA(ConcreteElementA elementA) {
        System.out.println("Visit concrete element a.");
    }

    @Override
    public void visitConcreteElementB(ConcreteElementB elementB) {
        System.out.println("Visit concrete element b.");
    }
}

// 抽象元素类
public abstract class AbstractElement {

    public abstract void accept(AbstractVisitor vistor);
}

// 具体元素类 A
public class ConcreteElementA extends AbstractElement {

    @Override
    public void accept(AbstractVisitor vistor) {
        vistor.visitConcreteElementA(this);
    }
}

// 具体元素类 B
public class ConcreteElementB extends AbstractElement {

    @Override
    public void accept(AbstractVisitor vistor) {
        vistor.visitConcreteElementB(this);
    }
}

// 对象结构类
public class ObjectStructure {

    private List<AbstractElement> elements = new ArrayList<>();

    public void attach(AbstractElement element) {
        elements.add(element);
    }

    public void detach(AbstractElement element) {
        elements.remove(element);
    }

    public void accept(AbstractVisitor visitor) {
        elements.forEach(element -> element.accept(visitor));
    }
}
```

#### 优缺点

| 优点 | 缺点 |
| :--- | :--- |
| 将变化的访问者封装, 符合单一职责原则 | 增加新的元素类困难 |
| 新增访问者符合开放关闭原则 | - |

#### 适用场景
- 对象结构中的元素类别很少改变的情况下, 希望对这些对象实施一些依赖于具体类型的操作

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)  
[访问者模式](https://blog.csdn.net/yanbober/article/details/45536787)
[访问者模式](https://blog.csdn.net/zhengzhb/article/details/7489639)
