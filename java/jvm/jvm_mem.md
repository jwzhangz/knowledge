# 内存
***
### 运行时数据区域
#### 程序计数器
当前线程所执行的字节码行号指示器。

由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式实现的，在任何一个确定的时刻，一个处理器(对于多核处理器来说是一个内核)都只会执行一条线程中的指令。  

每个线程都会有一个独立的程序计数器，这类内存称为“线程私有”内存。  

如果正在执行的方法是Native方法，这个计数器值为空(Undefined)。此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。
#### Java虚拟机栈  
#### 本地方法栈
#### Java堆
#### 方法区
#### 运行时常量池  
#### 直接内存  

## 对象内存布局
在HotSpot中，对象在内存中分为3块区域：对象头，实例数据，对其填充。  
### 对象头
对象头包括两部分信息，Mark Word 和类型指针。数组对象的对象头中还包括记录数组长度的数据。虚拟机可以通过Java对象的元数据信息确定Java对象的大小，但是无法从数组的元数据中确定数组的大小。
##### Mark Word
Mark Word在32位虚拟机和64位虚拟机中分别为32bit和63bit。  

##### 类型指针
即对象指向它的类元数据的指针，虚拟机通过这个指针确定这个对象是哪个类的实例。  

##### 对齐填充
HotSpot的自动内存管理系统要求对象起始地址必须是8字节的整数倍。

***
## 内存分配与回收策略

如果启动了本地线程分配缓冲，将按线程优先在TLAB上分配。

Serial/Serial Old收集器下的分配回收规则。

**新生代GC(Minor GC)**：发生在新生代的垃圾收集。Java对象大多数都朝生夕灭。Minor GC比较频繁，速度也比较快。

**老年代GC(Major GC/Full GC)**：老年代GC，Major GC经常伴随至少一次Minor GC(非绝对，Parallel Scavenge收集器的策略里就有直接进行Major GC的策略选择过程)。Major GC一般比Minor GC慢10倍以上。

```
-XX:+PrintGCDetails      参数则可以打印详细GC信息至控制台
-XX:+PrintGCDateStamps   则可以记录GC发生的详细时间
加入 -Xloggc:/home/XX/gc/app_gc.log 可以把GC输出至文件
```


###### 对象优先在Eden上分配
大多数情况下，对象在新生代Eden区中分配。当Eden区没有足够空间时，虚拟机将发起一次MinorGC。

###### 大对象直接进入老年代
大对象指需要连续内存空间的Java对象，最典型的就是很长的字符串以及数组。所以程序中尽量不要出现这种情况。

Serial和ParNew收集器可以设置 -XX:PretenureSizeThreshold 参数，大于这个值的对象直接在老年代分配。

###### 长期存活的对象将进入老年代
每个对象都有一个对象年龄计数器。如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor容纳，将被移动到Survivor中，对象年龄设为1。此后该对象每熬过一次Minor GC，年龄就增加1岁。当年龄增加到一定程度(默认为15)，就会被晋升到老年代中。通过 -XX:MaxTenuringThreshold 设置晋升老年代年龄阈值。

###### 动态对象年龄判定
为了能更好的适应不同程度的内存状况，虚拟机并不是永远的要求对象年龄必须达到MaxTenuringThreshold才能晋升老年代，如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代。

###### 空间分配担保
当出现大量对象在Minor GC后仍然存活的情况，Survivor无法容纳的对象直接进入老年代。

#### 内存回收
https://www.zhihu.com/question/41922036/answer/93079526

针对HotSpot VM的实现，它里面的GC其实准确分类只有两大种：
- Partial GC：并不收集整个GC堆的模式
  - Young GC：只收集young gen的GC
  - Old GC：只收集old gen的GC。只有CMS的concurrent collection是这个模式
  - Mixed GC：收集整个young gen以及部分old gen的GC。只有G1有这个模式
 
- Full GC：收集整个堆，包括young gen、old gen、perm gen（如果存在的话）等所有部分的模式。

最简单的分代式GC策略，按HotSpot VM的serial GC的实现来看，触发条件是：
- **young GC**：当young gen中的eden区分配满的时候触发。注意young GC中有部分存活对象会晋升到old gen，所以young GC后old gen的占用量通常会有所升高。
- **full GC**：当准备要触发一次young GC时，如果发现统计数据说之前young GC的平均晋升大小比目前old gen剩余的空间大，则不会触发young GC而是转为触发full GC（因为HotSpot VM的GC里，除了CMS的concurrent collection之外，其它能收集old gen的GC都会同时收集整个GC堆，包括young gen，所以不需要事先触发一次单独的young GC）；或者，如果有perm gen的话，要在perm gen分配空间但已经没有足够空间时，也要触发一次full GC；或者System.gc()、heap dump带GC，默认也是触发full GC。

HotSpot VM里其它非并发GC的触发条件复杂一些，不过大致的原理与上面说的其实一样。当然也总有例外。Parallel Scavenge（-XX:+UseParallelGC）框架下，默认是在要触发full GC前先执行一次young GC，并且两次GC之间能让应用程序稍微运行一小下，以期降低full GC的暂停时间（因为young GC会尽量清理了young gen的死对象，减少了full GC的工作量）。控制这个行为的VM参数是-XX:+ScavengeBeforeFullGC。这是HotSpot VM里的奇葩。
 
并发GC的触发条件就不太一样。以CMS GC为例，它主要是定时去检查old gen的使用量，当使用量超过了触发比例就会启动一次CMS GC，对old gen做并发收集。
 
### TLAB
https://www.jianshu.com/p/2343f2c0ecc4

TLAB的目的是在为新对象分配内存空间时，让每个Java应用线程能在使用自己专属的分配指针来分配空间，均摊对GC堆（eden区）里共享的分配指针做更新而带来的同步开销。也就是避免多线程在eden区申请内存时造成的同步开销。

###### TLAB私有，对象公有
TLAB只是让每个线程有私有的分配指针，但底下存对象的内存空间还是给所有线程访问的，只是其它线程无法在这个区域分配而已。

当一个TLAB用满（分配指针top撞上分配极限end了），就新申请一个TLAB，而在老TLAB里的对象还留在原地什么都不用管——它们无法感知自己是否是曾经从TLAB分配出来的，而只关心自己是在eden里分配的。

TLAB简单来说本质上就是三个指针：start，top 和 end （实际实现中还有一些额外信息但这里暂不讨论）。
 
###### TLAB refill
其中 start 和 end 是占位用的，标识出 eden 里被这个 TLAB 所管理的区域，卡住eden里的一块空间说其它线程别来这里分配了哈。而 top 就是里面的分配指针，一开始指向跟 start 同样的位置，然后逐渐分配，直到再要分配下一个对象就会撞上 end 的时候就会触发一次 TLAB refill。

###### TLAB refill包括下述几个动作
将当前TLAB抛弃（retire）掉。

将TLAB末尾尚未分配给Java对象的空间（浪费掉的空间）分配成一个假的“filler object”（目前是用int[]作为filler object）。这是为了保持GC堆可以线性parse（heap parseability）用的。

从eden新分配一块裸的空间出来

将新分配的空间范围记录到ThreadLocalAllocBuffer里

TLAB refill不成功（eden没有足够空间来分配这个新TLAB）就会触发YGC。


