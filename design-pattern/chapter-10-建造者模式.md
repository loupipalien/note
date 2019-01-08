### 建造者模式
将一个复杂对象的构建与它的表示分离, 使得同样的构建过程可以创建不同的表示; 建造者模式属于创建型模式

#### 模式角色
- Product: 产品类
- Director: 指挥者类
用于指挥产品的建造过程
- AbstractBuider: 抽象建造者类
- ConcreteBuilder: 具体建造者类

#### 代码示例
```
// 产品类
public class Product {
    List<String> parts = new ArrayList<>();

    public void add(String part) {
        parts.add(part);
    }

    public void show() {
        System.out.println(parts);
    }
}

// 指挥者类
public class Director {

    public Product build(AbstractBuilder builder) {
        builder.buildPartOne();
        builder.buildPartTwo();
        return builder.getResult();
    }
}

// 抽象建造者类
public abstract class AbstractBuilder {

    protected Product product = new Product();

    public abstract void buildPartOne();

    public abstract void buildPartTwo();

    public Product getResult() {
        return product;
    }
}

// 具体建造者类 A
public class ConcreteBuilderA extends AbstractBuilder {

    @Override
    public void buildPartOne() {
        product.add("A Part one.");
    }

    @Override
    public void buildPartTwo() {
        product.add("A Part two.");
    }

}

// 具体建造者类 B
public class ConcreteBuilderB extends AbstractBuilder {

    @Override
    public void buildPartOne() {
        product.add("B Part one.");
    }

    @Override
    public void buildPartTwo() {
        product.add("B Part two.");
    }

}
```

#### 优缺点

| 优点 | 缺点 |
| :--- | :--- |
| 客户端不必知道产品的内部组成细节, 将产品本身与产品的创建过程解耦, 使得相同的创建过程可以创建不同的产品对象 | 所创建产品一般具有较多的共同点, 其组成部分相似, 如果组成部分差异较大则不适合使用建造者模式 |
| 可以更加精细的控制产品的创建过程 | 如果产品的内部变化复杂, 可能导致需要定义许多具体建造者类 |
| 增加新的具体建造者无须修改原有代码, 符合开放关闭原则 | - |

#### 适用场景
- 需要生成的产品对象具有复杂的内部结构, 这些产品通常包含多个成员属性
- 需要生成的铲平对象的属性相互依赖, 需要指定其生成顺序
- 隔离复杂对象的创建, 并使得相同的创建过程可以创建不同的产品

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)   
[建造者模式](https://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/builder.html)
