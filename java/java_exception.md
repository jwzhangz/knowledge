# 异常
***
### Java的异常体系
**Throwable**: Java中所有异常和错误类的父类。只有这个类的实例（或者子类的实例）可以被虚拟机抛出或者被java的throw关键字抛出。同样，只有其或其子类可以出现在catch子句里面。  

**Error**: Throwable的子类，表示严重的问题发生了，而且这种错误是不可恢复的。  

**Exception**: Throwable的子类，应用程序应该要捕获其或其子类（RuntimeException例外），称为checked exception(除RuntimeException外的其他Exception异常)。比如：IOException, NoSuchMethodException...   

**RuntimeException**: Exception的子类，运行时异常，程序可以不捕获，称为unchecked exception。比如：NullPointException.  


