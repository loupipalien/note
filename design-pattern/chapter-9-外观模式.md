### 外观模式 (Facade Pattern)
为子系统中的一组接口提供一个一致的界面, 此模式定义了一个高层接口, 这个接口使得这一子系统更加容易使用; 外观模式属于结构型模式

#### 模式角色
- Facade: 外观类
- SubSystem: 子系统类

#### 代码示例
```
// 外观类
public class Facade {
    private SubSystemA systemA;
    private SubSystemB systemB;
    private SubSystemC systemC;

    public Facade() {
        systemA = new SubSystemA();
        systemB = new SubSystemB();
        systemC = new SubSystemC();
    }

    public void methodOne() {
        systemA.methodA();
        systemB.methodB();
    }

    public void methodTwo() {
        systemB.methodB();
        systemC.methodC();
    }
}

// 子系统类 A
public class SubSystemA {

    public void methodA() {
        System.out.println("Method A.");
    }
}

// 子系统类 B
public class SubSystemB {

    public void methodB() {
        System.out.println("Method B.");
    }
}

// 子系统类 C
public class SubSystemC {

    public void methodC() {
        System.out.println("Method C.");
    }
}
```

#### 优缺点

| 优点 | 缺点 |
| :--- | :--- |
| 对客户端屏蔽子系统组件, 减少了客户端处理的对象数目, 使得子系统使用起来更加容易 | 在不引入抽象外观类的情况下, 增加新的子系统可能需要修改外观类或客户端代码, 违背了开放关闭原则 |
| 实现了客户端和子系统之间松耦合的关系, 这使得子系统的组件变化不会影响到调用它的客户类, 只需要调整外观类即可 | 不能很好的限制客户端直接使用子系统 |
| 只是提供了一个访问子系统的统一入口, 并不影响用户直接使用子系统类 | - |

#### 适用场景
- 为一个复杂子系统和提供一个简单接口时
- 客户端程序与多个子系统之间存在很大的依赖性, 引入外观类将子系统与客户以及其他子系统解耦, 可以提高子系统的独立性和可移植性

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)
[外观模式](https://design-patterns.readthedocs.io/zh_CN/latest/structural_patterns/facade.html)
