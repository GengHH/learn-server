# learn-server
use ( tomcat + nginx + openJdk + mysql  ) to  configure linux server



##第一步：安装openjdk


-------------------
需要注意的是，java-1.8.0-openjdk包只包含Java运行时环境（Java Runtime Environment）。
如果是要开发Java应用程序，则需要安装java-1.8.0-openjdk-devel包。

注意：openJDK安装好后的目录位于：/usr/lib/jvm 下，包括jre和jdk。
      可以通过 rpm -ql java-1.8.0-openjdk  查看相关包的管理情况
------------------
###1.1.#安装openjdk
yum install java-1.8.0-openjdk


###1.2.#配置环境变量
vi /etc/profile
添加一下内容
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.111-0.b15.el6_8.x86_64

export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar  

export PATH=$PATH:$JAVA_HOME/bin

 
###1.3.#使文件生效
这样我们就设置好了JDK，再输入source /etc/profile 就可以生效了.  


##第二步：在linux的服务器中怎样使用mysql

前提 ：1、CentOs7版本默认情况下安装了mariadb-libs，必须先卸载才可以继续安装MySql。 
			a) 查找以前是否安装mariadb-libs
			># rpm -qa | grep -i mariadb-libs

			b)卸载已经安装的mariadb-libs
			># yum remove mariadb-libs-5.5.44-2.el7.centos.x86_64

	2、查找以前是否安装MySQL
			 ># rpm -qa | grep -i mysql
			 有的话，也删除
			 
###2.1、安装（**暂时不明白mysql客户端与mysql 服务器端两者的关系**）
	查看有没有安装过：
		  ># yum list installed mysql*
		  ># rpm -qa | grep mysql*

	查看有没有安装包：
		  ># yum list mysql*

	安装mysql客户端：
		  ># yum install mysql

	安装mysql 服务器端：
		  ># yum install mysql-server
		  ># yum install mysql-devel

###2.2、启动&&停止
 
	数据库字符集设置
		  mysql配置文件/etc/my.cnf中加入default-character-set=utf8

	启动mysql服务：
		  ># service mysqld start或者/etc/init.d/mysqld start
	开机启动：
		  ># chkconfig -add mysqld
	查看开机启动设置是否成功：
		  ># chkconfig --list | grep mysql*
		结果：mysqld    0:关闭    1:关闭    2:启用    3:启用    4:启用    5:启用    6:关闭
	停止：
		  ># service mysqld stop

	服务器直接关闭后mysql无法启动问题：
	chown -R mysql:mysql /var/lib/mysql
	/etc/rc.d/init.d/mysqld start

###2.3、登录数据库
 
	####2.3.1创建root管理员：
		  ># mysqladmin -u root password 123456

	####2.3.2登录：
		  ># mysql -u root -p 输入密码即可。
	####2.3.3忘记密码：
		  ># service mysqld stop
		  ># mysqld_safe --user=root --skip-grant-tables
		  ># mysql -u root
		  ># use mysql
		  ># update user set password=password("new_pass") where user="root";
		  ># flush privileges;  
 
###2.4、远程访问数据库 （**不常用**）
 
	开放防火墙的端口号
	mysql增加权限：mysql库中的user表新增一条记录host为“%”，user为“root”。


###2.5、Linux MySQL的几个重要目录
	####2.5.1数据库目录
		 /var/lib/mysql/
	####2.5.2配置文件
		 /usr/share /mysql（mysql.server命令及配置文件）
	####2.5.3相关命令
		 /usr/bin（mysqladmin mysqldump等命令）
	####2.5.4启动脚本
		 /etc/rc.d/init.d/（启动脚本文件mysql的目录）
		 

###2.6.进入数据库后
	####2.6.1.mysql> show databases 查看所有的数据库
	####2.6.2.mysql> use database_name 进入想要操作的数据库中
	####2.6.3.mysql> show tables 查看所有的数据表



###2.7.进入mysql的命令行界面后，使用语句
	source testDB.sql 来导入数据库



=========================================================================================================================

##第三步：centos 中安装nginx（反向代理服务器）
###3.1.# 我这里得到的最新版 nginx 是 v1.6.3-8.el7
	># yum list nginx
###3.2.# 安装 nginx
	># yum install nginx
###### 如果需要升级 nginx 运行 yum update nginx 即可

------------------------------------------
是啊，安装完了，可配置文件在哪啊？ 
在 Linux 系统中，我们可以使用 RPM (RedHat Package Manager)，
一种数据记录的方式来将你所需要的软件安装到你的Linux系统的一套管理机制，
来查找安装的软件包所在位置。
------------------------------------------


###3.3.# 查询 nginx 包名称
	># rpm -qa nginx
	nginx-1.6.3-9.el7.x86_64  
###3.4.# 查询 nginx-1.6.3-9.el7.x86_64 包文件路径
	># rpm -ql nginx-1.6.3-9.el7.x86_64
	...
	/etc/nginx/mime.types.default
	/etc/nginx/nginx.conf
	/etc/nginx/nginx.conf.default
	...

###3.5.#使用yun安装nginx后直接输入以下命令即可运行nginx
     （**如果使用的是编译安装法，则启动方式不同**）
      ># nginx

**会遇到的问题**：
centos6.5环境
修改nginx配置文件后，重启报错。（使用第5步时也会报这个错）：
nginx: [emerg] socket() [::]:80 failed (97: Address family not supported by protocol)
解决办法：
vim /etc/nginx/conf.d/default.conf
将
	listen       80 default_server;
	listen       [::]:80 default_server;

改成
	     listen       80;
	     #listen       [::]:80 default_server;

###3.6.#重启nginx
   ># nginx -s reload

###3.7.#关闭nginx

    #### 1）强制停止Nginx：
		pkill -9 nginx
    #### 2） 步骤1:查询nginx主进程号
		ps -ef | grep nginx
		在进程列表里 面找master进程，它的编号就是主进程号了。
		步骤2：发送信号
		快速停止Nginx：
		kill -TERM 主进程号
		
		
###3.8.# 修改过nginx.conf配置文件后，可以使用以下命令查看文件是否有问题

 	># nginx -t

	（文件无误）结果显示：
	nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
	nginx: configuration file /etc/nginx/nginx.conf test is successful
=====================================================================================================================


##第四步：配置tomcat

###（一）.使用源文件进行部署tomcat

####4.1.#下载和解压安装
-------------------------------------------------
可以在本地从官网上下载安装包，然后传到服务器
http://tomcat.apache.org/download-70.cgi
再进行解压，配置
-------------------------------------------------
	># cd /usr/local
	># wget https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.0.36/bin/apache-tomcat-8.0.36.tar.gz （该路径已失效）
	># tar xzf apache-tomcat-8.0.36.tar.gz
	># mv apache-tomcat-8.0.36 tomcat
	># ls
----------------------------------------------------------
个人习惯把tomcat放在/user/local下，下载后解压，再更名为tomcat 
---------------------------------------------------------

####4.2#.修改tomcat配置（如区项目名）
<host>下添加一行代码
  <Context docBase="blog" path="" debug="0"  reloadable="true"/>
  
  
####4.3#.修改配置文件conf/server.xml改为监听80端口，默认编码utf-8，并开启gzip压缩 (这一步不是必须的)
    <Connector port="80" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" executor="tomcatThreadPool" URIEncoding="utf-8"   
               compression="on"   
               compressionMinSize="50" noCompressionUserAgents="gozilla, traviata"   
               compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain" />
           <!-- A "Connector" using the shared thread pool-->
	
####4.4#.启动tomcat(一定进入tomcat的bin目录才执行的)

	># sh startup.sh   或者  ./startup.sh

####4.5#.关闭tomcat(一定进入tomcat的bin目录才执行的)

	># sh shutdown.sh  或者  ./shutdown.sh


####4.6#远程查看tomcat的控制台:
　　    进入tomcat/logs/文件夹下 键入指令：tail -f catalina.out 就可以查看控制台了
　　    或者
   	是使用bin目录下的catalina.sh run命令，如果能进入控制台，说明tomcat启动成功 （第二种方法好像行不通）
    	或者
	通过rpm -ql java-1.8.0-openjdk 查看相关的进程 ，来判断是不是启动成功了





##第五步：利用nginx整合tomcat


###5.1编辑安装目录下的nginx.conf，输入如下内容 :

			运行nginx所在的用户名和用户组
			#user  www www; 

			#启动进程数
			worker_processes 8;
			#全局错误日志及PID文件
			error_log  /usr/local/nginx/logs/nginx_error.log  crit;

			pid        /usr/local/nginx/nginx.pid;

			#Specifies the value for maximum file descriptors that can be opened by this process.

			worker_rlimit_nofile 65535;
			#工作模式及连接数上限
			events
			{
			  use epoll;
			  worker_connections 65535;
			}
			#设定http服务器，利用它的反向代理功能提供负载均衡支持
			http
			{
			  #设定mime类型
			  include       mime.types;
			  default_type  application/octet-stream;
			  include /usr/local/nginx/conf/proxy.conf;
			  #charset  gb2312;
			  #设定请求缓冲    
			  server_names_hash_bucket_size 128;
			  client_header_buffer_size 32k;
			  large_client_header_buffers 4 32k;
			  client_max_body_size 8m;

			  sendfile on;
			  tcp_nopush     on;

			  keepalive_timeout 60;

			  tcp_nodelay on;

			#  fastcgi_connect_timeout 300;
			#  fastcgi_send_timeout 300;
			#  fastcgi_read_timeout 300;
			#  fastcgi_buffer_size 64k;
			#  fastcgi_buffers 4 64k;
			#  fastcgi_busy_buffers_size 128k;
			#  fastcgi_temp_file_write_size 128k;

			#  gzip on;
			#  gzip_min_length  1k;
			#  gzip_buffers     4 16k;
			#  gzip_http_version 1.0;
			#  gzip_comp_level 2;
			#  gzip_types       text/plain application/x-javascript text/css application/xml;
			#  gzip_vary on;

			  #limit_zone  crawler  $binary_remote_addr  10m;
			 ###禁止通过ip访问站点
			  server{
				server_name _;
				return 404;
				}


			  server
			  {
			    listen       80;
			    server_name  localhost;
			    index index.html index.htm index.jsp;#设定访问的默认首页地址
			    root  /home/www/web/ROOT;#设定网站的资源存放路径

			    #limit_conn   crawler  20;    

			    location ~ .*.jsp$ #所有jsp的页面均交由tomcat处理
			    {
			      index index.jsp;
			      proxy_pass http://localhost:8080;#转向tomcat处理
			      }


			    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ #设定访问静态文件直接读取不经过tomcat
			    {
			      expires      30d;
			    }

			    location ~ .*\.(js|css)?$
			    {
			      expires      1h;
			    }    

			#定义访问日志的写入格式
			     log_format  access  '$remote_addr - $remote_user [$time_local] "$request" '
				      '$status $body_bytes_sent "$http_referer" '
				      '"$http_user_agent" $http_x_forwarded_for';
			    access_log  /usr/local/nginx/logs/localhost.log access;#设定访问日志的存放路径

			      }   
			}




## 第六步：linux Centos 6.5 安装桌面环境GNOME  (**感兴趣可以尝试**)

## 第七步：安装samba： (**感兴趣可以尝试**)
	yum -y install samba
	yum -y install sambaclient
	启动sumba服务 service samba start
	cd etc/init.d/smb restart
	cd etc/init.d/nmb restart
	测试配置：samba parm
	关闭防火墙：  service iptables stop
	centos 访问 smbclient //ip/share
	NT_ACCESS_DENIE :getenforce 0;
	window 访问 //ip/share

