# 异常
***
### Java的异常体系
**Throwable**: Java中所有异常和错误类的父类。只有这个类的实例（或者子类的实例）可以被虚拟机抛出或者被java的throw关键字抛出。同样，只有其或其子类可以出现在catch子句里面。  

**Error**: Throwable的子类，表示严重的问题发生了，而且这种错误是不可恢复的。  

**Exception**: Throwable的子类，应用程序应该要捕获其或其子类（RuntimeException例外），称为checked exception(除RuntimeException外的其他Exception异常)。比如：IOException, NoSuchMethodException.  


**RuntimeException**: Exception的子类，运行时异常，程序可以不捕获，称为unchecked exception。比如：NullPointException.  
 
从RuntimeException派生的异常在编译时不会被检查。  
已检查异常用于错误可被提前预知的情况。常见的错误包括输入输出，文件损坏，网络连接失败。  
未检查异常是程序员逻辑错误导致，不是不可避免的外部风险导致的。例如，引用null值。  

### 自定义异常
自定义异常时，最好提供两个构造函数，一个不带参数，另一个带消息字符串参数。  

### 声明已检查异常
可能抛出异常的方法必须在方法头声明一个throws语句:
``` Java
public void write(Objec obj, Srting filename)throws IOException, ReflectiveOperationException
```
可能会抛出已检查异常的任何方法都必须在方法头声明一个throws语句。  
覆盖一个方法时，它不能抛出比父类方法还要多的已检查异常。 
