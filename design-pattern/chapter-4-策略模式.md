### 策略模式 (Strategy Pattern)
定义了算法家族, 分别封装起来, 让它们之间可以互相替换, 此模式让算法的变化, 不会影响到使用算法的客户; 策略模式是一种类行为型模式

#### 模式角色
- Context: 上下文
- AbstractStrategy: 抽象策略
- ConcreteStrategy: 具体策略

#### 代码示例
```
// 上下文类
public class Context {
    private AbstractStrategy strategy;

    public void setStrategy(AbstractStrategy strategy) {
        this.strategy = strategy;
    }

    public int getResult(int a, int b) {
        return strategy.algorithm(a, b);
    }
}

// 抽象策略类
public abstract class AbstractStrategy {
    abstract int algorithm(int a, int b);
}

// 具体策略 A 类
public class ConcreteStrategyA extends AbstractStrategy {
    @Override
    int algorithm(int a, int b) {
        return a + b;
    }
}

// 具体策略 B 类
public class ConcreteStrategyB extends AbstractStrategy {
    @Override
    int algorithm(int a, int b) {
        return a * b;
    }
}
```

#### 优缺点

| 优点 | 缺点 |
| :--- | :--- |
| 提供了对 "开放关闭原则" 的支持 | 客户端必须了解所有的策略, 并自行决定使用哪一个策略类 |
| 提供了可以替换继承关系的办法 | 将造成很多策略类 |
| 提供了可以避免使用多重条件转移语句 | - |

#### 适用场景
- 一个系统里有许多类, 它们之间的区别在于它们的行为, 那么可以使用策略模式可以动态的让一个对象在许多行为中选择一种行为
- 不希望客端知道复杂的, 与算法相关的数据结构, 在具体策略类中封装算法和相关的数据结构, 提高算法的保密性和安全性

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)  
[策略模式](https://design-patterns.readthedocs.io/zh_CN/latest/behavioral_patterns/strategy.html)
