# bash小技巧
***
##### 当Bash用未声明变量时退出
``` Shell
set -o nounset
```
##### 当命令失败时让脚本退出
``` Shell
set -o errexit
```
###### Example：
``` Shell
#!/bin/bash

set -o xtrace
#set -o errexit  # 可以把这样注释掉看下执行效果有什么不一样。

echo "Before"
ls filenoexists  # ls也不存在的文件
echo "After"
```

##### 使用双引号来引用变量
有助于防止由于空格导致单词分割和由于识别和扩展了通配符导致的不必要匹配。

##### 用$(cmd)来做代换
命令代换 是用这个命令的输出结果取代命令本身。用$(cmd)而不是反引号`cmd`来做命令代换。

##### 用readonly来声明静态变量
``` Shell
readonly pw_file="/etc/file1"
```

##### 环境变量用大写字母命名，而自定义变量用小写
