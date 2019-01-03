### 简单工厂模式 (Simple Factory Pattern)
简单工厂模式又称为静态工厂方法 (Static Factory Method) 模式, 它属于类创建型模式; 在简单工厂模式中, 可以根据参数的不同返回不同类的实例, 简单工厂模式专门定义一个类负责创建其他类的实例, 被创建的实例通常都具有共同的父类; 简单工厂模式属于类创建型模式

#### 模式角色
- Factory: 工厂
工厂负责实现创建所有产品的内部逻辑
- AbstractProduct: 抽象产品
抽象产品是工厂所创建的所有产品的父类, 负责描述所有产品的公共接口
- ConcreteProduct: 具体产品
工厂所创建的产品的都是某个具体类的实例

#### 代码实例
```
// 工厂类
public class Factory {
    public static AbstractProduct create(String name) {
        AbstractProduct product = null;
        switch (name.toUpperCase()) {
            case "A":
                product = new ConcreteProductA();
                break;
            case "B":
                product = new ConcreteProductB();
                break;
            default:
                break;
        }
        return product;
    }
}

// 抽象产品类
public abstract class AbstractProduct {
    protected String name;

    abstract String getName();
}

// 具体产品 A 类
public class ConcreteProductA extends AbstractProduct {

    public ConcreteProductA(String name) {
        this.name = name;
    }

    @Override
    String getName() {
        return name;
    }
}

// 具体产品 B 类
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

#### 优缺点

| 优点 | 缺点    |
| :--- | :--- |
| 工厂类负责创建具体实例, 客户端无须关心如何创建 | 工厂类集中了所有产品的创建, 代码臃肿且不易维护 |
| 客户端需要不同实例时, 传递不同参数即可 | 由于使用了静态工厂方法, 静态方法不能被继承和重写, 会造成工厂角色无法继承 |
| 客户端可以引入配置文件, 可以在不改动代码的情况下更换具体产品类 | 在有新的产品增加时, 需要修改工厂类代码, 违反了开放关闭原则 |

#### 适用场景
适用于具体实现类较少时, 且不易变化的场景下; 以及作为某一个类的静态工厂, 例如 `java.text.DateFormat`

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)  
[简单工厂模式](https://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/simple_factory.html)
