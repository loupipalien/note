### 模板方法模式 (Template Method Pattern)
定义一个操作中的算法的骨架, 而将一些步骤延迟到子类, 模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤; 模板方法模式属于行为型模式

#### 模式角色
- AbstractClass: 抽象类
定义抽象的原语操作, 具体的子类将重定义它们以实现一个算法, 实现一个模板方法定义一个算法的骨架, 该模板方法不仅调用与原语操作也用于定义
- ConcreteClass: 具体子类
实现原语操作以完成算法中与特定子类相关的步骤

#### 代码示例
```Java
// 抽象类
public abstract class AbstractClass {
    public abstract void primitiveOperation();

    public void templateMethod() {
        primitiveOperation();
        System.out.println("Template method.");
    }
}

// 具体子类
public class ConcreteClass extends AbstractClass {
    @Override
    public void primitiveOperation() {
        System.out.println("Ptimitive opration");
    }
}
```

#### 优缺点

| 优点 | 缺点 |
| :--- | :--- |
| 提高了代码的复用性, 将相同部分的代码放在抽象的父类中 | 引入了抽象类, 每个不同的子类都需要一个子类来实现, 导致类的个数增加 |
| 扩展子类符合开放关闭原则 | - |
| 实现了反向控制, 父类调用新子类扩展的行为 | - |

#### 适用场景
- 一次性实现一个算法的不变部分, 并将可变的行为留给子类来实现
- 各子类中公共的行为应被提取出来并集中到一个公共父类中以避免代码重复

具体应用如 `java.util.HashMap` 与 `java.util.LinkedHashMap` 中的三个 `afterNodeXXX` 方法 

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)   
[模板方法模式](https://blog.csdn.net/carson_ho/article/details/54910518)  
[模板方法模式](https://blog.csdn.net/hguisu/article/details/7564039)
