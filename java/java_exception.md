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
这些异常要么是方法本身throw抛出的，要么是调用另一个带throws语句的方法。  

可以将多个子类合并到一个共同的父类。例如，某个方法抛出多个IOException的子类异常，将它们都在throws IOException中抛出。是否需要这样做，要取决于实际情况。  

覆盖一个方法时，它不能抛出比父类方法还要多的已检查异常。 

如果方法抛出一个无关的已检查异常，编译会报错。  

不能指定lambda表达式的异常类型。但是，如果lambda表达式会抛出一个异常，则只能将它传给一个声明了该异常的函数式接口的方法。例如，如下调用有错误：  
``` Java
list.forEach(obj -> write(obj, "output.dat"));
```
forEach方法的参数是函数式接口：
``` Java
public interface Consumer<T>{
    void accept(T t)
}
```
accept方法被声明为不抛出任何异常。  
[Java8（1）：当 Lambda 遇上受检异常](https://segmentfault.com/a/1190000007832130)

### 异常捕获
``` Java
try{
    ...
}catch(Exception1 e){
    ...
}catch(Exception2 e){
    ...
}
```
catch语句从上到下匹配，所以最具体的异常必须放在前面。  

也可以共享一个处理器
``` Java
try{
    ...
}catch(Exception1 | Exception2 e){
    ...
} 
```

### try-with-resources语句
格式：
``` Java
try (ResourceType1 res1 = init1; ResourceType2 res2 = init2; ResourceType3 res3 = init3; ){
    statements
}
```
每一个资源必须是实现AutoCloseable接口的类。当try块退出时，AutoCloseable接口的close方法被调用，将资源关闭。关闭资源的顺序与初始化顺序相反。  

如果try块中发生异常，则会先关闭资源，再传播异常。  
如果在关闭资源前出现异常，在关闭资源时又出现异常，则第一个异常更重要些，第二个异常作为一个"被抑制的"(suppressed)异常被附加到第一个异常上。可以通过ex.getSuppressed()获取。  

### finally子句
不管try语句正常完成还是发生异常，都会执行finally子句。  

应该避免在finally子句中抛出异常。如果try块中已经抛出异常，则try块的异常会被finally中的异常覆盖。  

finally中不应包含return语句。如果try块中也有一个return语句，那么它的返回值将会被finally中的return返回值覆盖。  

### 异常的重抛和链接


