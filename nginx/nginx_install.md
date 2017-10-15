###### 内核参数优化
修改 /etc/sysctl.conf
```
fs.file-max = 999999
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.ip_local_port_range = 1024    61000
net.ipv4.tcp_rmem = 4096 32768 262142
net.ipv4.tcp_wmem = 4096 32768 262142
net.core.netdev_max_backlog = 8096
net.core.rmem_default = 262144
net.core.wmem_default = 262144
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 1024
```
执行sysctl -p 命令使参数生效。
 
下载Nginx源码：
```
http://nginx.org/en/download.html
```
安装
```
tar -zxvf nginx-1.12.1.tar.gz
 
安装
./configure
make
make install
```

默认Nginx被安装到/usr/local/nginx/ 中，二进制文件路径为/usr/local/nginx/sbin/nginx，配置文件路径为/usr/local/nginx/conf/nginx.conf 。

###### 默认方式启动
直接执行二进制程序。
```
/usr/local/nginx/sbin/nginx
```
会读取默认路径下的配置文件 /usr/local/nginx/conf/nginx.conf 。也就是执行configure命令时使用的--conf-path=PATH指定的nginx.conf文件。

###### 指定配置文件启动
```
/usr/local/nginx/sbin/nginx -c /tmp/nginx.conf
```

###### 指定安装目录的方式启动
```
/usr/local/nginx/sbin/nginx -p /usr/local/nginx/
```

###### 显示版本
```
[root@linuxprobe nginx]# sbin/nginx -v
nginx version: nginx/1.12.1
```

###### 快速停止服务
```
sbin/nginx -s stop
```
强制停止Nginx。和 kill -s SIGTERM pid 效果是一样的。

###### 正常停止服务
```
sbin/nginx -s quit
```
正常处理完当前的请求再停止服务。

也可以用kill命令停止master进程和worker进程。
```
kill -s SIGQUIT <nginx master pid>
kill -s SIGWINCH <nginx worker pid>
```

###### 重新读取配置
```
sbin/nginx -s reload
```

###### 日志文件回滚
先把当前日志文件改名或备份，再重新打开就会生成新的日志文件。
```
sbin/nginx -s reopen
```
