### STS配置tomcat
###### 安装 tomcat
下载并解压 tomcat 压缩包。

系统变量中新建 TOMCAT_HOME 变量，在环境变量Path末尾加上 %TOMCAT_HOME%/bin ，用 ; 分隔。

###### 配置 tomcat
选择工具栏的Windows -> Preferences

在Preference页面，搜索框输入runtime，选择 Server -> Runtime Environments.点击 Add ，弹出对话框中选择 Apache Tomcat v9.0 。选中 Create a new local server。

![images/sts_add_tomcat](images\sts_add_tomcat.png)
