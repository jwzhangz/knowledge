

### 输入重定向
|符号|作用|
|--|--|
|命令 < 文件	   |将文件作为命令的标准输入  |
|命令 << 分界符	 |从标准输入中读入，直到遇见“分界符”才停止  |
|命令 < 文件1 > 文件2	|将文件1作为命令的标准输入并将标准输出到文件2  |

### 输出重定向
|符号|作用|
|--|--|
|命令 > 文件	|将标准输出重定向到一个文件中（清空原有文件的数据）|
|命令 2> 文件	|将错误输出重定向到一个文件中（清空原有文件的数据）|
|命令 >> 文件	|将标准输出重定向到一个文件中（追加到原有内容的后面）|
|命令 2>> 文件	|将错误输出重定向到一个文件中（追加到原有内容的后面）|
|命令 >> 文件 2>&1 或 命令 &>> 文件	|将标准输出与错误输出共同写入到文件中（追加到原有内容的后面）|

#### 例子
``` Bash
#重定向错误和标准输出到不同文件
ls -al test1 sss 2> test1 1> test2

#不显示输出信息
ls -al test1 sss &> /dev/null
```
