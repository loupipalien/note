### 错误处理

#### 使用异常而非返回码
在以前许多语言都不支持异常, 这些语言处理和汇报错误的手段都有限, 要么设置一个错误标识, 要么返回给调用者检查的错误码; 如以下代码所示
```
public class DeviceController {
    ...
    public void sendShutDown() {
        DeviceHandle handle =  getHandle(DEV1);
        // Check the state of the device
        if (handle != DeviceHandle.INVALID) {
            // Save the device status to the record field
            retrieveDeviceRecord(handle);
            // If not suspend, shut down
            if (record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle);
                clearDeviceWorkQueue(handle);
                closeDeivce(handle);
            } else {
                logger.log("Deivce suspend, Unable to shut down.");
            }
        } else {
            logger.log("Invild handle for: " + DEV1.toString());
        }
    }
}
```
这类手段的问题在于, 它们搞乱了调用者的代码; 调用者必须在调用后即刻检查错误, 不幸的是这个步骤很容易被遗忘; 所以遇到错误时, 最好抛出一个异常, 这样调用代码整洁, 其逻辑不会被错误处理搞乱; 如以下代码所示
```
public class DeviceController {
    ...
    public void sendShutDown() {
        try{
            tryToShutDown();
        } catch (DeviceShutDownError e) {
            logger.log(e);
        }
    }

    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDeivce(handle);
    }

    private DeviceHandle getHandle(DeviceID id) {
        ...
        throw new DeviceShutDownError("Invalid handle for: " + id.toString());
        ...
    }
    ...
}
```
这段代码整洁了很多, 是因为之纠结的两个元素设备关闭算法和错误处理现在被隔离了, 可以分别查看其中某一元素去理解它

#### 先写 Try-Catch-Finally 语句
异常的妙处之一就是, 它们在程序中定义了一个范围; 执行 try-catch-finally 语句中的 try 部分的代码时, 是在表明可以随时取消执行, 并在 catch 语句中接续; 在某种意义上, try 代码就像是事务, catch 代码将程序维持在一种持续状态, 无论 try 代码块中发生了什么均如此; 所以在编写可能抛出异常的代码时, 最好先写出 try-catch-finally 语句

#### 使用不可控异常
Java 中引入了可控异常, 即每个方法的签名都列出了它可能传递给调用者的异常, 而且这异常是方法类型的一部分, 如果签名与代码实际所做之事不符, 则代码在字面上就无法编译  
可控异常是个绝妙的主意, 而且它也是有所裨益的, 但现在业界已经清楚的是, 其对于强固软件的生产是非必需的; 另外一方面使用它也是有代价的, 可控异常的代价就是违反开放/闭合原则; 如果你在方法中抛出可控异常, 而 catch 语句在三个层级之上, 你就得在 catch 语句和抛出异常处之间的每个方法签名中声明该异常; 这意味着对软件中较低层级的修改, 都将波及较高层级的签名, 修改好的模块必须重新构建发布, 即便它们自身所关注的任何东西都没改动过  
如果在编写一套关键代码库, 则可控异常有时会有用, 因为必须捕获异常; 但对于一般的应用开发, 其依赖成本要高于收益

#### 给出异常发生的环境说明
抛出的每个异常都应当提供足够的环境说明, 以便判断错误的来源和分析; 在 Java 中, 可以从任何异常里得到堆栈踪迹 (stack trace), 然而堆栈信息中并没有该操作失败的初衷; 所以应创建信息充分的错误消息, 并和异常一起传递出去, 在消息中应该包括失败的操作和失败类型; 在应用中还应该使用日志在 catch 块中记录下来

#### 依调用者需要定义异常类
对错误的分类有很多种; 可以依其来源分类: 是来自组件还是其他地方? 或依其类型分类: 是设备错误还是编程错误? 在应用中定义异常类时, 最重要的考虑应该是它们如何被捕获; 以下是一个不太好的异常分类的例子
```
ACMEPort port = new ACMEPort(12);
try {
    port.open();
} catch (DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device Response exception", e);
} catch (ATM1212UnlockedEeception e) {
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch (GMXError e) {
    reportPortError(e);
    logger.log("Device response exception", e);
} finally {
    ...
}
```
改语句包含了一大堆异常捕获代码, 这并不出奇, 因为这时标准的处理流程: 捕获异常, 记录错误, 确保代码能继续工作; 我们可以通过打包调用 API, 确保它返回通用异常类型, 从而简化代码
```
LocalPort port = new LocalPort(12);
try {
    port.open();
} catch (PortDeviceFailure e) {
    reportPortError(e);
    logger.log(e.getMessage, e)
} finally {
    ...
}

public class LocalPort {
    private ACMEPort innerPort;

    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }

    public void open() {
      try {
          innerPort.open();
      } catch (DeviceResponseException e) {
          throw new PortDeviceFailure(e);
      } catch (ATM1212UnlockedEeception e) {
          throw new PortDeviceFailure(e);
      } catch (GMXError e) {
          throw new PortDeviceFailure(e);
      }
    }
}
```
LocalPort 类就是个简单的打包类, 捕获并翻译由 ACMEPort 类抛出的异常; 这种将第三方 API 打包是个良好的实践手段, 因为当你打包一个第三方 API, 就降低了对它的依赖: 未来可以不太痛苦的改用其他代码库

#### 定义常规流程
如果遵循了前文提及的建议, 在业务逻辑和错误处理代码之间就会有良好的区隔, 大量代码会开始变得像是整洁而简朴的算法; 以下代码是来自某个记账应用的开支总计模块
```
try {
    MealExpeneses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch (MealExpenesesNotFound e) {
    m_total += getMealPerDiem();
}
```
业务逻辑是, 如果消耗了餐食则计入总额中, 如果没有消耗则员工得到当日餐食补贴; 异常打断了业务逻辑, 如果不去处理特殊情况会不会好一点, 那样的代码看起来更简洁
```
MealExpeneses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```
如何把代码简洁化且符合业务逻辑, 答案是修改 ExpensesReportDAO, 使其总是返回 MealExpeneses 对象, 如果没有餐食消耗则返回一个餐食补贴的 MealExpeneses 对象
```
public class PerDiemMealExpenses implements MealExpeneses {
    public int getTotal() {
        // return the per diem default
    }
}
```
这种手法叫特例模式 (Special Case Pattern), 创建一个类或配置一个对象, 用来处理特例; 这样客户端代码就不用应付异常行为了, 异常行为被封装在了特例对象中

#### 别返回 null 值
返回 null 基本上就是在给自己增加工作量, 也是给调用者添乱, 只要一处没检查 null, 应用程序就会失控; 如果打算在方法中返回 null 值, 不如抛出异常, 或者返回特例对象; 如果是在调第三方 API 中可能返回 null 值的方法, 可以考虑用新方法打包这个方法, 在新方法中抛出异常或返回特例对象; 这样代码会更整洁, 也就能尽量避免 NullPointerException 的出现

#### 别传递 null 值
在方法中返回 null 值是糟糕的做法, 但将 null 值传递给其他方法就更糟糕了, 除非 API 要求你向它传递 null 值, 否则就要尽可能的避免传递 null 值; 传入 null 往往会得到错误, 但在大多数编程语言中, 没有良好的方法能对付调用者意外传入的 null 值; 恰当的做法就是禁止传入 null 值, 在编码的时候时时记住参数列表中 null 值意味着出问题了, 从而大量避免这种无心之失

#### 小结
整洁代码是可读的, 但也要加固; 可读和强固并不冲突, 如果将错误处理隔离看待, 独立于主要逻辑之外, 就能写出强固而整洁的代码
