在函数声明时，在其名字之前放上一个变量，就为该类型定义了一个方法。

例如：
```go
func (p Point) Distance(q Point) float64
```

附加的参数p，叫做方法的接收器(receiver). 这种p.Distance的表达式叫做选择器

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


通过嵌入结构体来扩展类型
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

常用的方法调用方式：
```go
p.Distance()
```

可以看成分为两步执行，p.Distance叫作“选择器”，选择器会返回一个方法"值"，这个值是将 Point.Distance 绑定到接收器变量的函数。这个函数可以被调用，而不必再指定接收器。
```go
p := Point{1, 2}
q := Point{4, 6}
distanceFromP := p.Distance
fmt.Println(distanceFromP(q))
```

#### 方法表达式
T是一个类型时，方法表达式可能会写作 T.f 或者 (*T).f ，会返回一个函数"值"。这个函数会将第一个参数作为接收器。
```go
p := Point{1, 2}
q := Point{4, 6}
distance := Point.Distance // method expression
fmt.Println(distance(p, q)) // "5"
fmt.Printf("%T\n", distance) // "func(Point, Point) float64"
```

#### 封装

一个对象的变量或者方法如果对调用方是不可见的话，一般就被定义为“封装”。
Go语言只有一种控制可见性的手段：大写首字母的标识符会从定义它们的包中被导出，小写字母的则不会。这种限制包内成员的方式同样适用于struct或者一个类型的方法。因而如果我们想要封装一个对象，我们必须将其定义为一个struct。

封装可以隐藏内部的实现，只暴露一些必要的接口，降低数据被修改的外部风险。


