### jps
```
命令格式
jps [ options ] [ hostid ]
jps可以通过RMI协议查询开启了RMI服务的远程虚拟机进程状态，hostid为RMI注册表中注册的主机名。

选项：
-q  只输出LVMID，省略主类的名称
-m  输出虚拟机进程启动时传递给主类main()函数的参数
-l  输出主类的全名，如果进程执行的是Jar包，输出Jar路径
-v  输出虚拟机进程启动时的JVM参数
```


### jstat
```
jstat 监视虚拟机各种运行状态信息
jstat [ option vmid [interval[s|ms] [count]] ]

interval和count代表时间间隔和次数，如果省略，默认只查询一次。
```
对于命令格式中的VMID和LVMID，如果是本地虚拟机进程，VMID和LVMID是一致的，如果是远程虚拟机进程，VMID应该是：
```
[protocal:][//]lvmid[@hostname[:port]/servername]
```

选项option：
```
-class            监视类装载、卸载数量、总空间以及类装载所耗费的时间
-gc               监视Java堆状况，包括Eden区，Survivor区、老年代等容量、已用空间、GC时间
-gccapacity       监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间
-gcutil           监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比
-gccause          与-gcutil功能一样，但是会额外输出导致上一次GC的原因
-gcnew            监视新生代GC状况
-gcnewcapacity    监视内容与-gcnew基本相同，输出主要关注使用到大最大、最小空间
-gcold            监视老年代GC
-gcoldcapacity    监视内容与-gcold基本相同，输出主要关注使用到的最大最小空间
-gcpermcapacity   输出永久带使用到的最大最小空间
-compiler         输出JIT编译器编译过的方法、耗时等信息
-printcompilation 输出已经被JIT编译的方法
```

### jinfo
实时查看和调整虚拟机各项参数。

使用 jps -v 可以查看虚拟机启动时显式指定的参数。如果想知道未被显式指定的参数的默认值，只能使用jinfo 的-flag选项查询。

可以使用 -sysprops 选项把虚拟机进程的System.getProperties()的内容打印出来。

可以使用-flag [+|-] name 或者 -flag name=value修改一部分运行期可写的虚拟机参数值。
```
例如：
jinfo -flag MaxPermSize 2788
```

### jmap： Java内存映像工具

生成堆转储快照(一般称为heapdump或者dump文件)。

获取Java堆转储快照，还有一些其他方法，通过 -XX:+HeapDumpOnOutOfMemoryError 参数，可以让虚拟机在OOM异常出现后自动生成dump文件，通过 -XX:+HeapDumpOnCtrlBreak参数则可以使用 Ctrl+Break键让虚拟机生成dump文件。或者在Linux系统下通过kill -3 命令发送进程退出信号“吓唬”一下虚拟机，也能拿到dump文件。

jmap很多功能在windows下受限。除了-dump选项和-histo所有操作系统都提供外，其余选项只能在Linux/Solaris下使用。

```
格式：
jmap [ option ] vmid

选项：
-dump          生成Java堆转储快照。格式为：-dump:[live,]format=b,file=<filename>,
               其中live子参数说明是否只dump出存活的对象
-finalizerinfo 显示在F-Queue中等待Finalizer线程执行finalize方法的对象。只在Linux/Solaris平台下有效
-heap          显示Java堆详细信息，如使用哪种回收器、参数配置、分代状况等。只在Linux/Solaris平台下有效
-histo         显示堆中对象统计信息，包括类、实例数量、合计容量
-permstat      以ClassLoader为统计口径显示永久带内存状况。只在Linux/Solaris平台下有效
-F             当虚拟机进程对-dump选项没有响应时，可使用这个选项强制生成dump快照。Linux/Solaris平台下有效
```

```
例子：
jmap -dump:format=b,file=eclipse.bin 3500
```

### jstack：Java堆栈跟踪工具
生成虚拟机当前时刻线程快照(一般称为threaddump或者javacore文件)。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合。

```
格式：
jstack [ option ] vmid

选项：
-F   当正常输出的请求不被响应时，强制输出线程堆栈
-l   除堆栈外，显示关于锁的附加信息
-m   如果调用到本地方法的话，可以显示c/c++的堆栈
```

Thread.getAllStackTraces() 方法可以在程序中获取线程栈。
```java
    for (Map.Entry<Thread, StackTraceElement[]> stackTrace : Thread.getAllStackTraces().entrySet()) {
        Thread thread = stackTrace.getKey();
        StackTraceElement[] stack = stackTrace.getValue();

        System.out.println("Thread: " + thread.getName());
        for (StackTraceElement element : stack) {
            System.out.println("\t" + element);
        }
    }
```
