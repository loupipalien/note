### 装饰模式 (Decorator Pattern)
动态的给一个对象添加一些额外的职责, 就增加功能来说, 装饰模式比生成子类更加灵活; 策略模式是一种结构型模式

#### 模式角色
- AbstractComponent: 抽象组件类
- ConcreteComponent: 具体构件类
- AbstractDecorator: 抽象装饰类
- ConcreteDecorator: 具体装饰类

#### 代码示例
```
// 抽象组件
public abstract class AbstractComponent {
    public abstract void operation();
}

// 具体构件
public class ConcreteComponent extends AbstractComponent {

    @Override
    public void operation() {
        System.out.println("Concrete component.");
    }
}

// 抽象装饰类
public abstract class AbstractDecorator extends AbstractComponent {
    protected AbstractComponent component;

    public void setComponent(AbstractComponent component) {
        this.component = component;
    }

    @Override
    public void operation() {
        if (component != null) {
            component.operation();
        }
    }
}

// 具体装饰类 A
public class ConcreteDecoratorA extends AbstractDecorator {

    @Override
    public void operation() {
        component.operation();
        System.out.println(addBehavior());
    }

    private String addBehavior() {
        return "ConcreteDecoratorA";
    }
}

// 具体装饰类 B
public class ConcreteDecoratorB extends AbstractDecorator {

    @Override
    public void operation() {
        component.operation();
        System.out.println(addBehavior());
    }

    private String addBehavior() {
        return "ConcreteDecoratorB";
    }
}
```

#### 优缺点

| 优点 | 缺点 |
| :--- | :--- |
| 装饰模式与继承模式的目的都是扩展对象的功能, 但是装饰模式可以提供比继承更多的灵活性 | 使用装饰模式进行系统设计时会产生很多装饰类, 这将增加系统复杂度, 加大学习与理解的难度 |
| 可以使用不同的具体装饰类以及这些装饰类的排列组合, 创造出更多不同行为的组合; 可以使用多个具体装饰类来装饰同一对象, 得到功能更强大的对象 | 这种比继承更加灵活的特性, 同时也意味着装饰模式比继承更加易于出错和更难排错 |
| 具体构件类与具体装饰类可以独立变化, 可以根据需要增加新的具体构建类和具体装饰类, 符合开放关闭原则 | - |

#### 使用场景
- 在不影响其他兑现对象的情况下, 以动态透明的方式给单个对象添加职责
- 当不能采用继承的方式对系统进行扩展或者采用继承不利于系统扩展和维护时; 不能采用给继承的情况主要有两类: 第一类是系统中存在大量独立的扩展, 为支持每一种组合将产生大量的子类, 使得子类数目呈爆炸性增长; 第二类是因为类定义为不能继承 (如 final 类)

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)   
[装饰模式](https://design-patterns.readthedocs.io/zh_CN/latest/structural_patterns/decorator.html)
