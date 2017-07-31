### Class文件结构
class文件是一组以8位字节为基础单位的二进制流，各数据项目严格按照顺序紧凑排列，中间没有分割符。如果数据项占用8位字节以上空间，则按照高位在前方式存储。也就是大端方式。

###### class文件格式

| 类型        | 名称           | 数量  |
| ------------- |-------------| -----|
|u4|magic|1|
|u2|minor_version|1|
|u2|major_version|1|
|u2|constant_pool_count|1|
|cp_info|constant_pool|constant_pool_count - 1|
|u2|access_flags|1|
|u2|this_class|1|
|u2|super_class|1|
|u2|interfaces_count|1|
|u2|interfaces|interfaces_count|
|u2|fields_count|1|
|field_info|fields|fields_count|
|u2|methods_count|1|
|method_info|methods|methods_count|
|u2|attributes_count|1|
|attribute_info|attributes|attributes_count|

###### 魔数
0xCAFEBABE

###### Class文件版本
5,6字节是次版本号。7,8字节是主版本号。Java版本号从45开始，从JDK1.1之后每个JDK大版本发布，主版本号向上加1。JDK1.2对应46。高版本JDK能向下兼容以前版本的class文件，但不能运行以后版本的class文件，即使文件格式没有变化，虚拟机也不会执行。

###### 常量池
常量池的容量计数从1开始，而不是0。其他集合类型从0开始计数。

常量池主要存放两大类常量：字面量和符号引用。字面量比较接近于Java语言层面的常量概念，比如文本字符串、声明为final的常量值等。符号引用属于编译原理方面的概念，包括三类常量：
+ 类和接口的全限定名
+ 字段的名称和描述符
+ 方法的名称和描述符

Java代码在进行Javac编译的时候，并不像C和C++那样有连接步骤，而是在虚拟机加载class文件时动态连接。也就是说，在class文件中不会保存各个方法、字段的最终内存布局信息。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析、翻译到具体的内存地址中。
