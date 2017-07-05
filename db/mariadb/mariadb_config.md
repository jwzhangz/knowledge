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
删除的时候也很简单，输入 mysqld.exe --remove MariaDB即可。
```
D:\dev\mariadb-10.2.6-winx64\bin>mysqld.exe --install MariaDB
Service successfully installed.
```
3.启动服务时遇到“系统错误1067”，在MariaDB解压后路径/data目录下找到“主机名.err ”，即可找到原因。  

4.CMD，切换到MariaDB解压后路径/bin,执行“mysql -uroot -p"切换至MariaDB模式，初始密码为空。Ctrl + c退出。  
再执行“SET PASSWORD FOR 'root'@'localhost' = PASSWORD('新密码');”，即可修改账户密码。  
修改密码：
```
mysqladmin.exe -u root password
New password: ********
Confirm new password: ********
```

```
MariaDB [(none)]> SHOW DATABASES;
```

```
MariaDB [(none)]> SHOW TABLES FROM mysql;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| column_stats              |
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| gtid_slave_pos            |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| host                      |
| index_stats               |
| innodb_index_stats        |
| innodb_table_stats        |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| roles_mapping             |
| servers                   |
| slow_log                  |
| table_stats               |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
30 rows in set (0.03 sec)
```
进入数据库
```
MariaDB [(none)]> use mysql
Database changed
MariaDB [mysql]>
```
显示表
```
MariaDB [mysql]> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| column_stats              |
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| gtid_slave_pos            |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| host                      |
| index_stats               |
| innodb_index_stats        |
| innodb_table_stats        |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| roles_mapping             |
| servers                   |
| slow_log                  |
| table_stats               |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
30 rows in set (0.00 sec)
```
显示user表的结构
```
MariaDB [mysql]> desc user;
```
刷新数据库
```
MariaDB [mysql]> flush privileges;
Query OK, 0 rows affected (0.13 sec)
```
查询user表中的host，user，password字段
```
MariaDB [mysql]> select host,user,password from user;
+-----------+------+-------------------------------------------+
| host      | user | password                                  |
+-----------+------+-------------------------------------------+
| localhost | root | *8232A1298A49F710DBEE0B330C42EEC825D4190A |
| 127.0.0.1 | root |                                           |
| ::1       | root |                                           |
| localhost |      |                                           |
+-----------+------+-------------------------------------------+
4 rows in set (0.00 sec)
```

### 创建数据库

```
MariaDB [(none)]> CREATE DATABASE mydb1;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb1              |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.00 sec)

MariaDB [(none)]> drop database mydb1;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> use mydb1
Database changed

```
### 创建表单
```
MariaDB [mydb1]> create table mybook (name char(15),price int,pages int);
Query OK, 0 rows affected (0.23 sec)

MariaDB [mydb1]> describe mybook;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| name  | char(15) | YES  |     | NULL    |       |
| price | int(11)  | YES  |     | NULL    |       |
| pages | int(11)  | YES  |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

### 管理表单及数据
#### 创建表
```
MariaDB [mydb1]> INSERT INTO mybook(name,price,pages) VALUES('linuxprobe','60',518);
Query OK, 1 row affected (0.05 sec)

MariaDB [mydb1]> select * from mybook;
+------------+-------+-------+
| name       | price | pages |
+------------+-------+-------+
| linuxprobe |    60 |   518 |
+------------+-------+-------+
1 row in set (0.00 sec)
```
#### 更新表
```
MariaDB [mydb1]> update mybook set price=55 ;
Query OK, 1 row affected (0.04 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
#### 删除表
```
MariaDB [mydb1]> delete from mybook;
Query OK, 1 row affected (0.03 sec)

MariaDB [mydb1]> select * from mybook;
Empty set (0.00 sec)
```




