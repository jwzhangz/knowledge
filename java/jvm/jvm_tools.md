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

### jmap
生成堆转储快照(一般称为heapdump或者dump文件)。
