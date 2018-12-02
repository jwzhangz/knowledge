接口类型是对类型行为的抽象

Go语言中接口类型的独特之处在于它是隐式实现的。也就是说，我们没有必要对于给定的具体类型定义所有满足的接口类型；简单地拥有一些必需的方法就足够了。

接口类型可以通过组合已经有的接口来定义。我们可以用这种方式以一个简写命名另一个接口，而不用声明它所有的方法。这种方式本称为接口内嵌。
```go
type ReadWriter interface {
    Reader
    Writer
}
```

或者甚至使用种混合的风格：
```go
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Writer
}
```

一个类型如果拥有一个接口需要的所有方法，那么这个类型就实现了这个接口。

接口的声明和赋值
```go
var w io.Writer
w = os.Stdout
```

一些方法的接收者是类型T本身，然而另一些则是一个 *T 的指针。在T类型的参数上调用一个 *T 的方法是合法的，只要这个参数是一个变量；编译器隐式的获取了它的地址。但这仅仅是一个语法糖：T类型的值不拥有所有 *T 指针的方法。


空接口
```go
interface{}
```

```go
var any interface{}
```

任意类型都可以被赋值给空接口类型变量。

any = true

#### 接口值
接口值，由两个部分组成，一个具体的类型和那个类型的值。

它们被称为接口的动态类型和动态值。

在Go语言中，变量总是被一个定义明确的值初始化，即使接口类型也不例外。对于一个接口的零值就是它的类型和值的部分都是nil。

一个接口值基于它的动态类型被描述为空或非空，所以这是一个空的接口值。可以通过使用w==nil或者w!=nil来判读接口值是否为空。调用一个空接口值上的任意方法都会产生panic:
```go
var w io.Writer
w.Write([]byte("hello")) // panic: nil pointer dereference
```

#### 接口的比较
接口值可以使用＝＝和！＝来进行比较。两个接口值相等仅当它们都是nil值或者它们的动态类型相同并且动态值也根据这个动态类型的＝＝操作相等。因为接口值是可比较的，所以它们可以用在map的键或者作为switch语句的操作数。

然而，如果两个接口值的动态类型相同，但是这个动态类型是不可比较的（ 比如切片） ，将它们进行比较就会失败并且panic

一个包含nil指针的接口不是nil接口

out 是 *bytes.Buffer 的空指针，动态值是nil。它的动态类型是*bytes.Buffer，意思就是out变量是一个包含空指针值的非空接口。
但是传入函数前是nil。
```go
import (
    "fmt"
    "io"
    "bytes"
)

func f(out io.Writer) {
    if out == nil{
        fmt.Printf("out == nil\n")
    }else{
        fmt.Printf("out != nil\n")
    }
    // ...do something...
    if out != nil {
        out.Write([]byte("done!\n"))
    }
}
const debug = false

func main() {    
    var buf *bytes.Buffer
    if debug {
        buf = new(bytes.Buffer) // enable collection of output
        fmt.Printf("new\n")
    } 
    if buf == nil{
        fmt.Printf("buf == nil\n")
    }else{
        fmt.Printf("buf != nil\n")
    }
    f(buf) // NOTE: subtly incorrect!
    if debug {
        fmt.Println("debug == true\n")
    }
}
```

输出：
```
buf == nil
out != nil
panic: runtime error: invalid memory address or nil pointer dereference
[signal 0xc0000005 code=0x1 addr=0x20 pc=0x49839d]
```

为什么传入函数前是nil，在函数中不是nil了呢？

#### error接口
```go
type error interface {
    Error() string
}
```


#### 类型断言 ？？？
```go
x.(T)
```

x表示一个接口类型

T表示一个接口类型或者实际动态类型

一个类型断言检查它操作对象的动态类型是否和断言的类型匹配。

第一种，如果断言的类型T是一个具体类型，类型断言检查x的动态类型是否和T相同。如果这个检查成功了，类型断言的结果是x的动态值，当然它的类型是T。如果检查失败，接下来这个操作会抛出panic。

第二种，如果断言的类型T是一个接口类型，类型断言检查是否x的动态类型满足T。如果这个检查成功了，动态值没有获取到；这个结果仍然是一个有相同类型和值部分的接口值，但是结果有类型T。
这块有点不明白

```go
package main

import (
    "os"
    "io"
    "fmt"
)

func main(){
    var w io.Writer
    w = os.Stdout
    fmt.Printf("%T\n", w)
    rw := w.(io.ReadWriter)
    fmt.Printf("%T\n", w)
    fmt.Printf("%T\n", rw)
}
```

```
output：
E:\code\go>go run test.go
*os.File
*os.File
*os.File
```

显示的都是实际的动态类型。怎么能够查看接口的类型？还是不重要，看怎么定义接口变量。

#### 类型开关

```go
switch x.(type) {
    case nil: // ...
    case int, uint: // ...
    case bool: // ...
    case string: // ...
    default: // ...
}
```

- 每个case有一到多个类型
- nil的case和if x == nil匹配
- default的case和如果其它case都不匹配的情况匹配
- 每一个case会被顺序的进行考虑，并且当一个匹配找到时，这个case中的内容会被执行。
- default case相对其它case的位置是无所谓的

类型开关语句有一个扩展的形式，它可以将提取的类型的值绑定到一个在每个case范围内的新变量。
```go
switch x := x.(type) { /* ... */ }
```

和一个switch语句相似地，一个类型开关隐式的创建了一个语言块，因此新变量x的定义不会和外面块中的x变量冲突。每一个case也会隐式的创建一个单独的语言块。

```go
func sqlQuote(x interface{}) string {
    switch x := x.(type) {
        case nil:
            return "NULL"
        case int, uint:
            return fmt.Sprintf("%d", x) // x has type interface{} here.
        case bool:
            if x {
                return "TRUE"
            } 
            return "FALSE"
        case string:
            return sqlQuoteString(x) // (not shown)
        default:
            panic(fmt.Sprintf("unexpected type %T: %v", x, x))
    }
}
```

在每个单一类型的case内部，变量x和这个case的类型相同。例如，变量x在bool的case中是bool类型和string的case中是string类型。

