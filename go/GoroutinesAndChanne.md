在Go语言中，每一个并发的执行单元叫作一个 goroutine 。

goroutine 和 线程的区别是什么？

程序启动时，main 函数运行在 main goroutine 中。

创建一个新的 goroutine：
```go
go f()
```

主函数返回时，所有的goroutine都会被直接打断，程序退出。除了从主函数退出或者直接终止程序之外，没有其它的编程方法能够让一个goroutine来打断另一个的执行。

go后跟的函数的参数会在go语句自身执行时被求值；

```go
go echo(c, input.Text(), 1*time.Second)
```

Go语言并没有提供在一个goroutine中终止另一个goroutine的方法。

#### Channels

goroutine 之间通过 channels 通信。channel 是有类型的，它的类型就是所发送数据的类型。

创建 channel 需要使用 make 函数

```go
//创建一个无缓存 int 类型的 channel
ch := make(chan int)
```

make 返回了一个底层数据结构的引用。当我们复制一个channel或用于函数参数传递时，我们只是拷贝了一个channel引用，因此调用者和被调用者将引用同一个channel对象。和其它的引用类型一样，channel的零值也是nil。

对一个nil的channel发送和接收操作会永远阻塞。

两个相同类型的channel可以使用==运算符比较。如果两个channel引用的是相同的对象，那么比较的结果为真。一个channel也可以和nil进行比较。

channel 发送和接收数据
```go
//将x的值发送到 channel 中
ch <- x
//将 channel 中的值读取到x中
x = <- ch
//读取 channel 中的值， 但是抛弃
<- ch
```

关闭 channel
```go
close(ch)
```

向一个已经关闭的 channel 发送数据将导致panic异常。

重复关闭一个channel将导致panic异常，试图关闭一个nil值的channel也将导致panic异常。

对一个已经 close 的 channel 的接收操作依然能够接收到之前已经成功发送的数据。如果 channel 中已经没有数据的话将产生一个零值的数据。 


##### 无缓存和带缓存的 channel

创建一个带缓存的 channel
```go
ch = make(chan int, 3)
//无缓存的channel
ch = make(chan int, 0)
```

一个基于无缓存Channels的发送操作将导致发送者 goroutine 阻塞，直到另一个goroutine在相同的Channels上执行接收操作，当发送的值通过Channels成功传输之后，两个goroutine可以继续执行后面的语句。反之，如果接收操作先发生，那么接收者 goroutine 也将阻塞，直到有另一个goroutine在相同的Channels上执行发送操作。


有时候会用无缓存的 channel 来实现两个 goroutine 的同步。

```go
//一般用发送空的结构体来实现通知
done <- struct{}{}
```

##### 带缓存的 channel

创建一个缓存为3个元素的 channel
```go
ch = make(chan string, 3)
```

向缓存Channel的发送操作就是向内部缓存队列的尾部插入元素，接收操作则是从队列的头部删除元素。

如果内部缓存队列是满的，那么发送操作将阻塞直到因另一个goroutine执行接收操作而释放了新的队列空间。

相反，如果channel是空的，接收操作将阻塞直到有另一个goroutine执行发送操作而向队列插入元素。

```go
channel 的容量指缓存的元素的最大数量
//获取容量
cap(ch)
channel 的长度指缓存队列中有效的元素个数
//获取长度
len(ch)
```

##### goroutines泄漏

如果 goroutines 在读写 channel 时被阻塞住，并且之后也不会再操作 channel ，那么 goroutines 将不会被回收，从而造成 goroutines 泄露。

解决办法就是创建一个合适大小的 buffered channel ，使写入操作不会出现阻塞。

##### 接收方判断 channel 关闭
1. 通过返回值
```go
x, ok := <-ch
if !ok {
}
```

2. range 循环，当channel被关闭并且没有值可接收时跳出循环。
```go
for x := range ch
```

不管一个channel是否被关闭，当它没有被引用时将会被Go语言的垃圾自动回收器回收。

试图重复关闭一个channel将导致panic异常。

试图关闭一个nil值的channel也将导致panic异常。


##### channel 的方向

当一个channel作为一个函数参数时，它一般总是被专门用于只发送或者只接收。

Go语言的类型系统提供了单方向的channel类型，分别用于只发送或只接收的channel。

chan<- int 表示一个只发送int的channel，只能发送不能接收。

<-chan int 表示一个只接收int的channel，只能接收不能发送。
这种限制将在编译期检测。

只有在发送者所在的goroutine才会调用close函数(即使不是由该 goroutine 创建)，对一个只接收的channel调用close将是一个编译错误。

##### channel 的隐式转换

channel 创建时是双向的，当以双向 channel 为参数调用函数 f 时，将发生隐式转换，双向 channel 转换为 chan<-int 类型。单向 channel 无法转换为双向 chennel。
```go
func f(out chan<- int)
```

##### select 语句

有时候我们希望能够从channel中发送或者接收值，并避免因为发送或者接收导致的阻塞，尤其是当channel没有准备好写或者读时。select语句就可以实现这样的功能。

每一个case代表一个通信操作(在某个channel上进行发送或者接收)并且会包含一些语句组成的一个语句块。接收操作也可以不用赋值。

也可以没有 default 分支。

```go
select {
case <-ch1:
// ...
case x := <-ch2:
// ...use x...
case ch3 <- y:
// ...
default:
// ...
}
```

select会等待case中有能够执行的case时去执行。当条件满足时，select才会去通信并执行case之后的语句；这时候其它通信是不会执行的。

一个没有任何case的select语句写作select{}，会永远地等待下去。

如果多个case同时就绪时，select会随机地选择一个执行。

对一个nil的channel发送和接收操作会永远阻塞。

并发 goroutine 的退出

通过关闭一个 channel 来通知其他的 goroutine 退出。其他的 goroutine 轮询这个退出 channel。

```go
var done = make(chan struct{})
func cancelled() bool {
    select {
        case <-done:
            return true
        default:
            return false
    }
}
```



