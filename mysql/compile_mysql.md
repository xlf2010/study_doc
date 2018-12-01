ubuntu 14.4-lts compile mysql-5.7.20
1. 安装编译工具 gcc,cmake,make等，在执行cmake会检查所需的依赖，如果依赖缺失的话，一般按提示执行命令即可，如果找不到指定的包，则可下载源码编译安装。然后编译时参数指定安装目录即可。
````
apt-get install <dependency> 
````
2. 下载源码链接：<a>https://dev.mysql.com/downloads/mysql/</a>
3. 解压下载的源码, 
````
tar -zxf mysql-5.7.20.tar.gz 
````
4. 进入源码目录执行 ,-DWITH_DEBUG=1 表示开启调试模式，默认为关闭
````	
cmake . -DWITH_DEBUG=1
````
配置时提示找不到boost_1_59_0，可以添加参数下载
````
cmake . -DWITH_DEBUG=1 -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/tmp/
````
配置成功后执行make编译安装，make install 默认安装到 /usr/local/mysql/
````
	make
````
5. 安装完成后，执行添加mysql用户,以mysql用户执行服务,可以不配置密码
````
useradd mysql
````
6. 新增配置文件 conf/my.cnf
```
	[client]
	default-character-set=utf8
	auto-rehash
	# socket=/usr/local/mysql/mysql.sock
	
	[mysqld]
	datadir=/usr/local/mysql/data
	socket=/tmp/mysql.sock
	user=mysql
	# Disabling symbolic-links is recommended to prevent assorted security risks
	symbolic-links=0
	character-set-server=utf8
	max_allowed_packet=10M
	performance_schema=ON
	sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
	
	[mysqld_safe]
	log-error=/usr/local/mysql/logs/localhost.localdomain.err
	pid-file=/usr/local/mysql/mysqld.pid
```
7. 初始化mysql服务,此时会生成root密码，注意记下，等会登录时用到
````
./bin/mysqld --initialize --user=mysql
````
8.启动mysql服务
````
	su -c "/usr/local/mysql/bin/mysqld_safe --defaults-file=/usr/local/mysql/config/my.cnf > /dev/null"
````