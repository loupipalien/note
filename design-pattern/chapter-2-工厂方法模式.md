### 工厂方法模式 (Factory Method Pattern)
定义一个用于创建对象的接口, 让子类决定实例化哪一个类, 使一个类的实例化延迟到其子类; 工厂方法模式属于类创建型模式

#### 模式角色
- AbstractFactory: 抽象工厂
抽象工厂是所有具体工厂的父类, 负责描述所有工厂的公共接口
- ConcreteFactory: 具体工厂
负责实现创建对应的产品的工厂
- AbstractProduct: 抽象产品
抽象产品是工厂所创建的所有产品的父类, 负责描述所有产品的公共接口
- ConcreteProduct: 具体产品
工厂所创建的产品的都是某个具体类的实例

#### 代码实例
```
public abstract class AbstractFactory {
    abstract AbstractProduct create();
}

public class ConcreteFactoryA extends AbstractFactory {

    @Override
    ConcreteProductA create() {
        return new ConcreteProductA("A");
    }
}

public class ConcreteFactoryB extends AbstractFactory {

    @Override
    ConcreteProductA create() {
        return new ConcreteProductA("B");
    }
}


public abstract class AbstractProduct {
    protected String name;

    abstract String getName();
}

public class ConcreteProductA extends AbstractProduct {

    public ConcreteProductA(String name) {
        this.name = name;
    }

    @Override
    String getName() {
        return name;
    }
}

public class ConcreteProductB extends AbstractProduct {

    public ConcreteProductB(String name) {
        this.name = name;
    }

    @Override
    String getName() {
        return name;
    }
}
```

#### 工厂方法模式的优缺点

| 优点 | 缺点    |
| :--- | :--- |
| 符合开放关闭原则, 新增产品时只需增加具体的产品类和对应具体工厂即可 | 增加具体产品类时也得增加具体工厂类, 类的个数成倍增加 |
| 符合单一职责原则, 每个具体工厂类只负责创建对应的产品 | 在客户端使用抽象层进行定义, 增加了系统的抽象性和理解难度 |
| 不使用静态工厂方法, 可以形成继承的等级结构 | 虽然保证了系统的可扩展性, 但是对于客户端来说, 在更换新产品时也需要修改客户端代码 |

工厂模式方法是简单工厂模式的进一步的抽象和推广, 由于使用了多态性, 工厂方法模式保持了简单工厂模式的优点, 而且克服了它的缺点, 当然也带来类成倍增加等的问题

#### 适用场景
当一个类希望通过子类来指定创建对象时, 在工厂方法模式中, 对于抽象工厂类只需要提供一个创建产品的接口, 而由其子类来确定具体要创建的对象, 利用面向对象的多态性和里氏代换原则, 在程序运行时, 子类对象将覆盖父类对象, 从而使得系统更容易扩展

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)  
[工厂方法模式](https://blog.csdn.net/carson_ho/article/details/52343584)  
[工厂方法模式](https://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/factory_method.html)
