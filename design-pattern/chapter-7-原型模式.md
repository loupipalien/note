### 原型模式 (Prototype Pattern)
用原型实例指定创建对象的种类, 并并且通过拷贝这些原型创建新的对象; 原型模式属于对创建型模式

#### 模式角色
- - AbstractPrototype: 抽象原型
抽象原型是所有具体原型的父类, 负责描述所有原型的抽象类
- ConcretePrototype: 具体原型
负责实现创建对应的原型

#### 代码示例
```Java
// 抽象原型类
public abstract class AbstractPrototype {
    private String id;

    public AbstractPrototype(String id) {
        this.id = id;
    }

    public String getId() {
        return id;
    }

    public abstract Prototype clone();
}

// 具体原型类 A
public class ConcretePrototypeA extends AbstractPrototype {
    public ConcretePrototypeA(String id) {
        super(id);
    }

    public Prototype clone() {
        return new ConcretePrototypeA(this.getId());
    }
}

// 具体原型类 B
public class ConcretePrototypeB extends AbstractPrototype {
    public ConcretePrototypeB(String id) {
        super(id);
    }

    public Prototype clone() {
        return new ConcretePrototypeB(this.getId());
    }
}
```

#### 优缺点

| 优点 | 缺点 |
| :--- | :--- |
| 使用原型模式创建对象比直接 new 一个对象在性能上要好的多 | 原型模式最主要的缺点是每一个类都必须配备一个克隆方法 |
| 使得创建对象就像我们在编辑文档时的复制粘贴一样简单 | - |

#### 适用场景
- 重复地创建相似对象时可以考虑使用原型模式
>**参考:**
- [大话设计模式](https://book.douban.com/subject/2334288/)
- [原型模式](https://juejin.im/post/5b48bec15188251b3d79bf9c)
