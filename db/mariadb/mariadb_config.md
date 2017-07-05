### 下载安装
下载zip版，解压缩到指定目录。


配置文件一般放在你的安装目录内，名为my.ini 。文件夹中，一般包含5个MySQL自带的配置文件，my- small.ini、my-medium.ini、my-large.ini、my-huge.ini和my-innodb-heavy-4G.ini，请你根据自己机器的内存大小，他们是my.ini的模板配置文件。拷贝一份，改名为my.ini.

修改[mysqld] 和[client]段中的端口，以及字符集如下  
```
[client]  
#password = [your_password]  
port  = 3306  
socket  = /tmp/mysql.sock  
default-character-set=utf8   
# The MariaDB server  
#  
[mysqld]  
port  = 3306  
socket  = /tmp/mysql.sock  
character-set-server=utf8 
```

