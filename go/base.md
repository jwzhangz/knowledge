###### 下载docker源码
git clone https://github.com/docker/docker.git

[docker 源码分析 四（基于1.8.2版本），Docker镜像的获取和存储](https://www.cnblogs.com/yuhan-TB/p/5053370.html)  
[Docker源码分析（一）：Docker架构](http://www.infoq.com/cn/articles/docker-source-code-analysis-part1?utm_source=infoq&utm_campaign=user_page&utm_medium=link)  
[docker源码编译安装步骤解析](http://blog.csdn.net/wujianyongw4/article/details/70598722?locationNum=13&fps=1)


###### main函数
main函数保存在名为main的包里。如果不声明main包，构建工具将不会生成可执行文件。
```go
package main
```

### 声明导入项
```go
import(
    //从标准库导入包，只需要给出包名，编译器会去环境变量设置的路径中查找
    "log"
    "os"
    
    _"code/sample/matchers"
    "code/sample/search"
)
```

所有处于同一文件夹里的代码文件，必须使用同一包名。按照惯例，包和文件夹同名。

为了可读性，go编译器不允许声明导入某个包却不使用。有时不引用包的内容，却需要运行包中的init函数，就在导入路径前加下划线。

init函数在main函数之前执行。


###### 变量定义
```go
var value = 0;
```

变量没有定义在函数作用域内，会作为包级变量。变量名小写字母开头。

包中的标识符分为公开和非公开。以大写字母开头的标识符为公开标识符，以小写字母开头的为非公开标识符。非公开标识符不能被其他包中的代码直接访问。

###### 初值
所有变量会被初始化为其零值。数值类型是0；字符串是空字符串；布尔类型是false；指针是nil。

###### 迭代
for  range 关键字对数组，字符串，切片，映射和通道迭代。每次迭代返回两个值，第一个是索引，第二个是元素值的副本。

下划线起占位符的作用。

```go
for _, feed := range feeds {
	// Retrieve a matcher for the search.
	matcher, exists := matchers[feed.Type]
	if !exists {
		matcher = matchers["default"]
	}

	// Launch the goroutine to perform the search.
	go func(matcher Matcher, feed *Feed) {
		Match(matcher, feed, searchTerm, results)
		waitGroup.Done()
	}(matcher, feed)
}
```

###### goroutine
go 关键字启动一个goroutine，是个并发执行的过程。

下面的函数是个匿名函数。

```go
go func(matcher Matcher, feed *Feed) {
	Match(matcher, feed, searchTerm, results)
	waitGroup.Done()
}(matcher, feed)
```
