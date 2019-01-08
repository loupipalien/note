### 命令模式 (Command Pattern)
将一个请求封装为一个对象, 从而使可以用不同的请求对客户进行参数化, 对请求排队或记录请求日志, 以及支持可撤销的操作; 命令模式属于行为型模式

#### 模式角色
- Invoker: 调用者类
- AbstractCommand: 抽象命令类
- ConcreteCommand: 具体命令类
- Receiver: 接受者类

#### 代码示例
```
// 调用者类
public class Invoker {

    private AbstractCommand command;

    public void setCommand(AbstractCommand command) {
        this.command = command;
    }

    public void executeCommand(AbstractCommand command) {
        command.execute();
    }
}

// 抽象命令类
public abstract class AbstractCommand {

    protected Receiver receiver;

    public void setReceiver(Receiver receiver) {
        this.receiver = receiver;
    }

    public abstract void execute();
}

// 具体命令类
public class ConcreteCommand extends AbstractCommand {

    @Override
    public void execute() {
        receiver.operation();
    }
}

// 接受者类
public class Receiver {

    public void operation() {
        System.out.println("operation.");
    }
}
```

#### 优缺点

| 优点 | 缺点 |
| :--- | :--- |
| 降低系统耦合度 | 使用命令模式可能会导致有过多的具体命令类 |
| 新命令的加入符合开放关闭原则 | - |
| 可以方便的实现对请求的 Undo 和 Redo | - |

#### 适用场景
- 系统需要将请求调用者和请求接收者解耦, 使得调用者和接收者不直接交互
- 系统需要在不同的时间指定请求, 将请求排毒和执行请求
- 系统需要支持命令的撤销操作和恢复操作

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)  
[命令模式](https://design-patterns.readthedocs.io/zh_CN/latest/behavioral_patterns/command.html)
