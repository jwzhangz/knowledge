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
basedir=D:/JavaE/DB/mariadb-10.0.8-winx64  
datadir=D:/JavaE/DB/mariadb-10.0.8-winx64/data  
character-set-server=utf8 
```
### 启动服务

1.在开始菜单输入cmd，右击以管理员身份运行，运行后将执行目录cd到你的问价解压目录中的bin目录中，例如我的解压在D:\JavaE\DB\mariadb-10.0.8-winx64： 
输入 D: 回车  
输入 cd D:\JavaE\DB\mariadb-10.0.8-winx64\bin 回车即可跳转到bin目录。  
2.完成上面的步骤之后就可以开始安装服务和启动服务了。  
输入 mysqld.exe --install MariaDB  
等待成功后，输入 net start MariaDB 即可启动服务开始你的MariaDB之旅了。 
如果需要停止该服务，输入 net stop MariaDB 即可停止服务  

```
D:\dev\mariadb-10.2.6-winx64\bin>mysqld.exe --install MariaDB
Service successfully installed.
```
3.启动服务时遇到“系统错误1067”，在MariaDB解压后路径/data目录下找到“主机名.err ”，即可找到原因。  

4.CMD，切换到MariaDB解压后路径/bin,执行“mysql -uroot -p"切换至MariaDB模式，初始密码为空。Ctrl + c退出。  
再执行“SET PASSWORD FOR 'root'@'localhost' = PASSWORD('新密码');”，即可修改账户密码。  
修改密码：mysqladmin.exe -u root password  



