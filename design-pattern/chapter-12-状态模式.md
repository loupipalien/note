### 状态模式
当一个对象的内存状态改变时允许改变其行为, 这个对象看起来就像是改变了其类; 状态模式属于行为型模式

#### 模式角色
- Context: 上下文类
- AbstractState: 抽象状态类
- ConcreteState: 具体状态类

#### 代码示例
```
// 上下文类
public class Context {
    private AbstractState state;

    public void changeState(AbstractState state) {
        this.state = state;
    }

    public void request() {
        state.handle(this);
    }
}

// 抽象状态类
public abstract class AbstractState {

    public abstract void handle(Context context);
}

// 具体状态类 A
public class ConcreteStateA extends AbstractState {

    @Override
    public void handle(Context context) {
        System.out.println("Concrete State A.");
    }
}

// 具体状态类 B
public class ConcreteStateB extends AbstractState {

    @Override
    public void handle(Context context) {
        System.out.println("Concrete State B.");
    }
}
```

#### 优缺点

| 优点 | 缺点 |
| :--- | :--- |
| 封装了转换规则 | 增加了系统类的个数 |
| 可以方便的增加新的类, 只需要改变对象状态即可改变对象的行为 | 状态模式的结构与实现都较为复杂, 使用不当将导致程序结构和代码的混乱 |
| 将状态转换逻辑与状态对象合成一体, 而不是巨大的条件语句块 | 状态模式对 "开放封闭原则" 支持不友好, 增加新的状态类需要修改负责状态转换的源代码, 修改某个状态类的行为也需要修改对应类的源代码 |

#### 适用场景
- 对象的行为依赖于它的状态属性, 并且可以依据它的状态改变而改变相关行为
- 代码中包含大量与对象状态有关的条件语句, 当这些条件语句导致代码的可维护性和灵活性变差时

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)   
[状态模式](https://design-patterns.readthedocs.io/zh_CN/latest/behavioral_patterns/state.html)
