###### 块配置项
由一个块配置项名和一对大括号组成。
```
events{
    ...
}

http{
    upstream backend{
        127.0.0.1:8080;
    }
    
    gzip on;
    server {
        ...
        location /webstatic{
            gzip off;
        }
    }
}
```

块配置项可以嵌套。内层块直接继承外层块，例如，上例中，server块里的任意配置都是基于http块已有配置的。当内层块和外层块中的配置发生冲突时，究竟以谁为准，取决于解析这个块的模块。

###### 配置项的语法格式
```
配置项名 配置项值1 配置项值2 ...;
```
行首是配置项名，这些配置项名必须是Nginx的某一个模块想要处理的，否则Nginx会认为配置文件出现了非法的配置项名。配置项名和配置项值以空格分割。

配置项值可以是数字，字符串或者正则表达式，一个配置项可以有一个或多个值，取决于解析的模块，配置项值用空格分割。

如果配置项值的字符串中包含空格，则需要在首尾加单引号或者双引号。

每行末尾要加分号。

###### 注释
用 '#' 注释一行配置。

###### 配置项的单位
```
K/k 千字节
M/m 兆字节

ms 毫秒
s  秒
m  分钟
h  小时
d  天
w  周
M  月(包含30天)
y  年(包含365天)
```

###### 在配置项中使用变量
在变量名前加 $ 符号引用变量。只有少数模块支持变量，不是通用的。

***

# Nginx 服务基本配置

### 用于调试和定位问题的配置项

###### error日志
```
语法： error_log /path/file level;
默认： error_log logs/error.log error
```
文件也可以是/dev/null，这样就不会产生日志了。如果/path/file是stderr，日志会输出到标准错误文件中。

level日志输出级别包括：debug, info, notice, warn, error, crit, alert, emerg，从作至右一次增大。当设置一个级别时，大于该级别的日志会被输出。

*如果日志级别设置为debug，必须在configure时加入--with-debug配置项。*

### 正常运行配置项
###### 定义环境变量
```
env VAR=VALUE;
```

###### 嵌入其他配置文件
```
include /path/file;
```
文件名可以带通配符。

###### pid文件路径
```
pid path/file;
默认： pid logs/nginx.pid
```
保存master进程ID的pid文件存放路径。默认与configure执行时的参数'--pid-path'所指定的路径是相同的，也可以随时修改，但应确保Nginx有权限在相应的目标中创建pid文件。该文件直接影响Nginx是都可以运行。

###### Nginx worker进程运行的用户和用户组
```
user username [groupname];
默认：user nobody nobody
```
user用于设置master进程启动后，fork出的worker进程运行在哪个用户和用户组下。当按照"user username;"设置时，用户组名与用户名相同。

若用户在configure命令执行时使用了参数 --user=username和--group=groupname，此时nginx.conf将使用参数中指定的用户和用户组。

###### 设置Nginx worker进程可以打开的最大句柄描述符数
```
worker_rlimit_nofile limit;
```
设置一个worker进程可以打开的最大句柄数。

###### 限制信号队列
```
worker_rlimit_sigpending limit;
```
设置每隔用户发往Nginx的信号队列的大小。当某个用户的信号队列满了，再发送的信号量会被丢掉。


### 优化性能配置项
###### Nginx worker进程个数
```
worker_processes number;
默认： worker_processes 1
```
在master/worker方式下，定义worker进程的个数。

每个worker进程都是单线程进程。

###### 绑定Nginx worker进程到指定的CPU内核
```
worker_cpu_affinity cpumask [cpumask]

例如，如果有4颗CPU内核，就可以这样配置：
worker_processes 4
worker_cpu_affinity 1000 0100 0010 0001;
```

该配置仅对Linux系统有效。

### 事件类配置项

###### 是否打开accept锁
```
accept_mutex [on|off]
默认：accept_mutex on;
```
accept_mutex是Nginx的负载均衡锁。

###### 使用accept锁后到真正建立连接之间的延迟时间
```
accept_mutex_delay Nms;
默认： accept_mutex_delay 500ms;
```
同一时间只有一个worker进程能够取到accept锁。不是阻塞锁，取不到会立刻返回。如果一个worker进程没有获取到accept锁，则需要等至少accept_mutex_delay时间后才能再次试图获取锁。

###### 批量建立新连接
```
multi_accept [on|off]
默认： multi_accept off;
```
当事件模型通知有新连接时，尽可能对本次调度中客户端发起的所有TCP请求都建立连接。

###### 选择事件模型
```
use [kqueue|rtsig|epoll|/dev/poll|select|poll|eventport];
默认： Nginx会自动使用最合适的事件模型。
```

###### 每个worker的最大连接数
```
worker_connections number;
```
定义每个worker进程可以同时处理的最大连接数。

***
# 配置一个静态Web服务器

静态Web服务器的主要功能由ngx_http_core_module模块实现。

一个典型的静态Web服务器还会包含多个server块和location块，
