###### 将标准输出重定向到一个文件的同时并在屏幕上显示
输出标准输出和标准错误，同时保存到文件logfile  
```
<command> 2>&1 | tee <logfile>
[root@home root]# id das 2>&1 |tee logfile
id: das: No such user
[root@home root]# cat logfile
id: das: No such user
```

注释：管道的作用为把一个进程的标准输出作为另一个进程的标准输入。2>&1是把标准错误重定向到标准输出的副本一起输出。上面的命令，把标准输出和标准错误都输出作为tee命令的标准输入，tee的作用为把标准输入的内容拷贝到文件，并输出。
