#### 函数声明
```go
func name(parameter-list) (result-list) {
    body
}
```

定义一个类型为 func(int) int 的变量f
```go
var f func(int) int
```

函数的参数通过传值方式传递，函数的形参是实参的拷贝。对形参进行修改不会影响实参。
如果实参包括引用类型，如指针，slice(切片)、map、function、channel等类型，实参可能会由于函数的间接引用被修改。



如果一组形参或返回值有相同的类型，我们不必为每个形参都写出参数类型。下面2个声明是等价的：
```go
func f(i, j, k int, s, t string) { /* ... */ }
func f(i int, j int, k int, s string, t string) { /* ... */ }
```

如果函数返回一个无名变量或者没有返回值，返回值列表的括号是可以省略的。
```go
func hypot(x, y float64) float64 {
    return math.Sqrt(x*x + y*y)
}
```

返回值也可以像形式参数一样被命名。在这种情况下，每个返回值被声明成一个局部变量，并根据该返回值的类型，将其初始化为其零值。 

如果一个函数在声明时，包含返回值列表，该函数必须以 return语句结尾。

```go
func add(x int, y int) int {return x + y}
func sub(x, y int) (z int) { z = x - y; return}
func first(x int, _ int) int { return x }
func zero(int, int) int { return 0 }
```

如果一个函数将所有的返回值都显示的变量名，那么该函数的return语句可以省略操作数。这称之为bare return。不宜过度使用 bare return，会使代码难以阅读。

函数的类型被称为函数的标识符。如果两个函数形式参数列表和返回值列表中的变量类型一一对应，那么这两个函数被认为有相同的类型和标识符。形参和返回值的变量名不影响函数标识符也不影响它们是否可以以省略参数类型的形式表示。

Go语言没有默认参数值，也没有任何方法可以通过参数名指定形参。 这点和Python不一样。

在函数体中，函数的形参作为局部变量，被初始化为调用者提供的值。函数的形参和有名返回值作为函数最外层的局部变量，被存储在相同的词法块中。

你可能会偶尔遇到没有函数体的函数声明，这表示该函数不是以Go实现的。这样的声明定义了函数标识符。
```go
package math
func Sin(x float64) float //implemented in assembly language
```

一个函数可以有多个返回值。
如果某个值不被使用，可以将其分配给blank identifier:
```go
links, _ := findLinks(url) // errors ignored
```

还可以这样调用
```go
log.Println(findLinks(url))
和下面是等价的
links, err := findLinks(url)
log.Println(links, err)
```

#### 函数的错误处理

1. 返回 bool 类型值
```go
value, ok := cache.Lookup(key)
if !ok {
    // ...cache[key] does not exist…
}
```

2. error 接口类型。

如果错误值为nil，则函数运行正常。
```go
resp, err := http.Get(url)
if err != nil{
    return nil, err
}
```

错误信息应提供清晰的从原因到后果的因果链，
```
genesis: crashed: no parachute: G-switch failed: bad relay orientation
```

由于错误信息经常是以链式组合在一起的，所以错误信息中应避免大写和换行符。最终的错误信息可能很长，我们可以通过类似grep的工具处理错误信息。
编写错误信息时，我们要确保错误信息对问题细节的描述是详尽的。尤其是要注意错误信息表达的一致性，即相同的函数或同包内的同一组函数返回的错误在构成和处理方式上是相似的。

#### 函数值
在Go中，函数被看作第一类值（ first-class values） ：函数像其他值一样，拥有类型，可以被赋值给其他变量，传递给函数，从函数返回。对函数值（ function value） 的调用类似函数调用。
```go
f := square
fmt.Println(f(3)) // "9"
```

函数类型的零值是nil，调用值为nil的函数值会引起panic错误

函数值可以与nil比较，但函数值之间是不可比较的，也不能用函数值作为map的key。为什么？
```go
var f func(int) int{
    if f != nil {
        f(3)
    }
}
```


#### 匿名函数
拥有函数名的函数只能在包级语法块中被声明，通过函数字面量（ function literal） ，我们可绕过这一限制，在任何表达式中表示一个函数值。

函数字面量的语法和函数声明相似，区别在于func关键字后没有函数名。称为匿名函数。

```go
func squares() func() int {
    var x int
    return func() int {
        x++
        return x * x
    }
}
func main() {
    f := squares()
    fmt.Println(f()) // "1"
    fmt.Println(f()) // "4"
    fmt.Println(f()) // "9"
    fmt.Println(f()) // "16"
}
```

Go使用闭包（closures）技术实现函数值，Go程序员也把函数值叫做闭包。

函数值不仅仅是一串代码，还记录了状态。匿名函数可以是外部函数中的变量的。因此，这个函数中保存了外部函数中变量的状态。因此函数是引用类型并且不能比较。
还是有点不明白，以后再弄清楚。

#### 循环变量的引用问题
下面代码中的问题：
```go
var rmdirs []func()
for _, dir := range tempDirs() {
    os.MkdirAll(dir, 0755)
    rmdirs = append(rmdirs, func() {
        os.RemoveAll(dir) // NOTE: incorrect!
    })
}
```

for循环定义了一个词法块，循环变量dir在这个词法块中定义。也就是说，循环中定义的所有函数共享同一个变量dir。并且这个dir还会被for循环不停地更改。
匿名函数直接引用的是变量的内存，而不是复制变量的值。

在循环中定义一个新的变量可以解决这个问题：
```go
for _, dir := range tempDirs() {
    dir := dir // declares inner dir, initialized to outer dir
    // ...
}
```

#### 可变参数
在声明可变参数函数时，需要在参数列表的最后一个参数类型之前加上省略符号“...”，这表示这个位置的参数对应着任意数量的该类型参数。
```go
func sum(vals...int) int {
```

在函数体中,vals被看作是类型为[] int的切片。sum可以接收任意数量的int型参数：
```go
fmt.Println(sum())           // "0"
fmt.Println(sum(3))          // "3"
fmt.Println(sum(1, 2, 3, 4)) // "10"
```

调用者隐式的创建一个数组，并将原始参数复制到数组中，再把数组的一个切片作为参数传给被调函数。
如果原始参数已经是切片类型，我们该如何传递给sum？只需在最后一个参数后加上省略符。
```go
values := []int{1, 2, 3, 4}
fmt.Println(sum(values...)) // "10"
```

#### Deferred函数
在调用普通函数或方法前加上关键字defer 
当defer语句被执行时，跟在defer后面的函数会被延迟执行。直到包含该defer语句的函数执行完毕时，defer后的函数才会被执行，不论包含defer语句的函数是通过return正常结束，还是由于panic
导致的异常结束。你可以在一个函数中执行多条defer语句，它们的执行顺序与声明顺序相反。

defer语句经常被用于处理成对的操作，如打开、关闭、连接、断开连接、加锁、释放锁。通过defer机制，不论函数逻辑多复杂，都能保证在任何执行路径下，资源被释放。释放资源的defer应该直接跟在请求资源的语句后。

也可以用于记录函数的进入，退出。

例如：
```go
func ReadFile(filename string) ([]byte, error) {
    f, err := os.Open(filename)
    if err != nil {
        return nil, err
    } 
    defer f.Close()
    return ReadAll(f)
}
```

利用defer 实现 trace 函数的功能
```go
func trace(msg string) func() {
    start := time.Now()
    log.Printf("enter %s", msg)
    return func() {
        log.Printf("exit %s (%s)", msg,time.Since(start))
    }
}

defer trace("bigSlowOperation")()
```

#### defer 记录返回值
defer语句中的函数会在return语句更新返回值变量后再执行，又因为在函数中定义的匿名函数可以访问该函数包括返回值变量在内的所有变量，所以，对匿名函数采用defer机制，可以使其观察函数的返回值。
```go
func double(x int) (result int) {
    defer func() { fmt.Printf("double(%d) = %d\n", x,result) }()
    return x + x
}
```

#### painc异常
当panic异常发生时，程序会中断运行，并立即执行在该goroutine中被延迟的函数（ defer 机制）。

直接调用内置的panic函数也会引发panic异常,panic函数接受任何值作为参数

#### 输出堆栈信息
runtime包允许程序员输出堆栈信息。在下面的例子中，我们通过在main函数中延迟调用printStack输出堆栈信息。
```go
func main() {
    defer printStack()
    f(3)
} 

func printStack() {
    var buf [4096]byte
    n := runtime.Stack(buf[:], false)
    os.Stdout.Write(buf[:n])
}
```


