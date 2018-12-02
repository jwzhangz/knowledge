在函数声明时，在其名字之前放上一个变量，就为该类型定义了一个方法。

例如：
```go
func (p Point) Distance(q Point) float64
```

附加的参数p，叫做方法的接收器(receiver)

这种p.Distance的表达式叫做选择器

在Go语言中，我们并不会像其它语言那样用this或者self作为接收器；我们可以任意的选择接收器的名字。

由于接收器的名字经常会被使用到，所以保持其在方法间传递时的一致性和简短性是不错的主意。这里的建议是可以使用其类型的第一个字母，比如这里使用了Point的首字母p。


#### 基于指针的方法
```go
func (p *Point) ScaleBy(factor float64)
```

这里的括号是必须的；没有括号的话这个表达式可能会被理解为 *(Point.ScaleBy) 

想要调用指针类型方法 (*Point).ScaleBy ，只要提供一个Point类型的指针即可

```go
r := &Point{1, 2}
r.ScaleBy(2)
```

如果接收器p是一个Point类型的变量，并且其方法需要一个Point指针作为接收器，我们可以用下面这种简短的写法：
```go
p.ScaleBy(2)
```

编译器会隐式地帮我们用&p去调用ScaleBy这个方法。这种简写方法只适用于“变量”，包括struct里的字段比如p.X，以及array和slice内的元素比如perim[0]。我们不能通过一个无法取到地址的接收器来调用指针方法，比如临时变量的内存地址就无法获取得到：
```go
Point{1, 2}.ScaleBy(2) // compile error: can't take address of Point literal
```

可以用一个 *Point 这样的接收器来调用Point的方法，编译器在这里也会给我们隐式地插入 * 这个操作符，所以下面这两种写法等价的
```go
func (p Point) Distance(q Point) float64
pptr.Distance(q)
(*pptr).Distance(q)
```


如果命名类型T的方法是用T类型自己来做接收器，那么拷贝这种类型的实例就是安全的；调用他的任何一个方法也就会产生一个值的拷贝。

如果一个方法使用指针作为接收器，原始对象和拷贝对象实际上指向同一个对象。

就像一些函数允许nil指针作为参数一样，方法理论上也可以用nil指针作为其接收器，尤其当nil对于对象来说是合法的零值时，比如map或者slice。


#### 通过嵌入结构体来扩展类型
```go
type Point struct{ X, Y float64 }
type ColoredPoint struct {
    Point
    Color color.RGBA
}
```

ColoredPoint 类型可以直接引用 Point类型中的元素和方法。
```go
cp.X = 1
var p = ColoredPoint{Point{1, 1}, red}
p.ScaleBy(2)
```

注意 ColoredPoint 不是 Point 的子类。

一个struct类型也可能会有多个匿名字段。我们将ColoredPoint定义为下面这样：

```go
type ColoredPoint struct {
    Point
    color.RGBA
}
```

当编译器解析一个选择器到方法时，比如p.ScaleBy，它会首先去找直接定义在这个类型里的ScaleBy方法，然后找被ColoredPoint的内嵌字段们引入的方法，然后去找Point和RGBA的内嵌字段引入的方法，然后一直递归向下找。如果选择器有二义性的话编译器会报错，比如你在同一级里有两个同名的方法。

#### 方法值和方法表达式



