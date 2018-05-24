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
