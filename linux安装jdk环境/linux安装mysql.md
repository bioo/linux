# linux安装mysql
####转发https://www.cnblogs.com/shizhongyang/p/8464876.html

linux安装mysql详细步骤
最近买了个腾讯云服务器，搭建环境。

该笔记用于系统上未装过mysql的干净系统第一次安装mysql。自己指定安装目录，指定数据文件目录。

linux系统版本： CentOS 7.3 64位

安装源文件版本：mysql-5.7.21-linux-glibc2.12-x86_64.tar.gz

mysql安装位置：/software/mysql

数据库文件数据位置：/data/mysql

 

注：未防止混淆，这里都用绝对路径执行命令

        除了文件内容中的#,这里所有带#都是linux命令

　　>mysql 是mysql的命令

 

步骤：

1、在根目录下创建文件夹software和数据库数据文件/data/mysql

#mkdir /software/

#mkdir /data/mysql

 

2、上传mysql-5.7.21-linux-glibc2.12-x86_64.tar.gz文件到/software下

 

#cd /software/

#tar -zxvf mysql-5.7.21-linux-glibc2.12-x86_64.tar.gz

 

3、更改解压缩后的文件夹名称

#mv /software/mysql-5.7.21-linux-glibc2.12-x86_64/  /software/mysql

 

4、创建mysql用户组和mysql用户

#groupadd mysql

#useradd -r -g mysql mysql

 

5、关联myql用户到mysql用户组中

#chown -R mysql:mysql  /software/mysql/

#chown -R mysql:mysql  /data/mysql/

#chown -R mysql  /software/mysql/

#chown -R mysql  /data/mysql

 

6、更改mysql安装文件夹mysql/的权限

#chmod -R 755 /software/mysql/

 

7、安装libaio依赖包，由于我买的腾讯云服务器centos系统自带的有这个依赖包所以不需要安装，不过自带的依赖包会报错，后面介绍解决办法

查询是否暗转libaio依赖包

#yum search libaio

如果没安装，可以用下面命令安装

#yum install libaio

 

8、初始化mysql命令

#cd /software/mysql/bin

#./mysqld --user=mysql --basedir=/software/mysql --datadir=/data/mysql --initialize

在执行上面命令时特别要注意一行内容   

[Note] A temporary password is generated for root@localhost: o*s#gqh)F4Ck

root@localhost: 后面跟的是mysql数据库登录的临时密码，各人安装生成的临时密码不一样

如果初始化时报错如下：

error while loading shared libraries: libnuma.so.1: cannot open shared objec

是因为libnuma安装的是32位，我们这里需要64位的，执行下面语句就可以解决

#yum install numactl.x86_64

执行完后重新初始化mysql命令

 

9、启动mysql服务

# sh /software/mysql/support-files/mysql.server start

上面启动mysql服务命令是会报错的，因为没有修改mysql的配置文件，报错内容大致如下：

./support-files/mysql.server: line 239: my_print_defaults: command not found
./support-files/mysql.server: line 259: cd: /usr/local/mysql: No such file or directory
Starting MySQL ERROR! Couldn't find MySQL server (/usr/local/mysql/bin/mysqld_safe)
 
 
10、修改Mysql配置文件
#vim /software/mysql/support-files/mysql.server
修改前
if test -z "$basedir"
then
basedir=/usr/local/mysql
bindir=/usr/local/mysql/bin
if test -z "$datadir"
then
datadir=/usr/local/mysql/data
fi
sbindir=/usr/local/mysql/bin
libexecdir=/usr/local/mysql/bin
else
bindir="$basedir/bin"
if test -z "$datadir"
then
datadir="$basedir/data"
fi
sbindir="$basedir/sbin"
libexecdir="$basedir/libexec"
fi

修改后

if test -z "$basedir"
then
basedir=/software/mysql
bindir=/software/mysql/bin
if test -z "$datadir"
then
datadir=/data/mysql
fi
sbindir=/software/mysql/bin
libexecdir=/software/mysql/bin
else
bindir="$basedir/bin"
if test -z "$datadir"
then
datadir="$basedir/data"
fi
sbindir="$basedir/sbin"
libexecdir="$basedir/libexec"
fi

保存退出

#cp /software/mysql/support-files/mysql.server  /etc/init.d/mysqld
#chmod 755 /etc/init.d/mysqld
11、修改my.cnf文件

#vi /etc/my.cnf

将下面内容复制替换当前的my.cnf文件中的内容

[client]
no-beep
socket =/software/mysql/mysql.sock
# pipe
# socket=0.0
port=3306
[mysql]
default-character-set=utf8
[mysqld]
lower_case_table_names=0 #针对linux会导致大小写区分问题（0:大小写敏感;1:大小写不敏感）
basedir=/software/mysql
datadir=/data/mysql
port=3306
pid-file=/software/mysql/mysqld.pid
#skip-grant-tables
skip-name-resolve
socket = /software/mysql/mysql.sock
character-set-server=utf8
default-storage-engine=INNODB
explicit_defaults_for_timestamp = true
# Server Id.
server-id=1
max_connections=2000
query_cache_size=0
table_open_cache=2000
tmp_table_size=246M
thread_cache_size=300
#限定用于每个数据库线程的栈大小。默认设置足以满足大多数应用
thread_stack = 192k
key_buffer_size=512M
read_buffer_size=4M
read_rnd_buffer_size=32M
innodb_data_home_dir = /data/mysql
innodb_flush_log_at_trx_commit=0
innodb_log_buffer_size=16M
innodb_buffer_pool_size=256M
innodb_log_file_size=128M
innodb_thread_concurrency=128
innodb_autoextend_increment=1000
innodb_buffer_pool_instances=8
innodb_concurrency_tickets=5000
innodb_old_blocks_time=1000
innodb_open_files=300
innodb_stats_on_metadata=0
innodb_file_per_table=1
innodb_checksum_algorithm=0
back_log=80
flush_time=0
join_buffer_size=128M
max_allowed_packet=1024M
max_connect_errors=2000
open_files_limit=4161
query_cache_type=0
sort_buffer_size=32M
table_definition_cache=1400
binlog_row_event_max_size=8K
sync_master_info=10000
sync_relay_log=10000
sync_relay_log_info=10000
#批量插入数据缓存大小，可以有效提高插入效率，默认为8M
bulk_insert_buffer_size = 64M
interactive_timeout = 120
wait_timeout = 120
log-bin-trust-function-creators=1
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

 

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d

保存退出

 

12、启动mysql

#/etc/init.d/mysqld start

新版本的安装包会报错，错误内容如下：

Starting MySQL.Logging to '/data/mysql/SZY.err'.
2018-07-02T10:09:03.779928Z mysqld_safe The file /usr/local/mysql/bin/mysqld
does not exist or is not executable. Please cd to the mysql installation
directory and restart this script from there as follows:
./bin/mysqld_safe&
See http://dev.mysql.com/doc/mysql/en/mysqld-safe.html for more information
ERROR! The server quit without updating PID file (/software/mysql/mysqld.pid).

因为新版本的mysql安全启动安装包只认/usr/local/mysql这个路径。

解决办法：

方法1、建立软连接

例 #cd /usr/local/mysql

#ln -s /sofware/mysql/bin/myslqd mysqld

 

方法2、修改mysqld_safe文件（有强迫症的同学建议这种，我用的这种）

# vim /software/mysql/bin/mysqld_safe

将所有的/usr/local/mysql改为/software/mysql

保存退出。（可以将这个文件拷出来再修改然后替换）

 

13、登录mysql

#/software/mysql/bin/mysql -u root –p

 

14、输入临时密码。临时密码就是第8条root@localhost:后面的内容

 

15、修改mysql的登录密码

>mysql   set password=password('root');

>mysql  grant all privileges on *.* to root@'%' identified by 'root';

>mysql flush privileges;

 

16、完成，此时mysql的登录名root  登录密码root

17.解决MySQL登录ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using passwor)问题

	编辑/etc/my.cnf,然后在里面找到 [mysqld] 这一项，然后在该配置项下添加 skip-grant-tables 这个配置，然后保存文件。 
	
	重启mysql服务```service mysqld restart```
	
	再次进入MySQL命令行 /software/mysql/bin/mysql -u root -p,输入密码时直接回车，就会进入MySQL数据库了，这个时候按照常规流程修改root密码即可。
	如果还是提示错误，则输入 /software/mysql/bin/mysql -u root 即可。
	
18.MySQL5.7更改密码时出现ERROR 1054 (42S22): Unknown column 'password' in 'field list'
	
	新安装的MySQL5.7，登录时提示密码错误，安装的时候并没有更改密码，后来通过免密码登录的方式更改密码，输入update mysql.user  set password=password('root') where user='root'时提示ERROR 1054 (42S22): Unknown column 'password' in 'field list'，原来是mysql数据库下已经没有password这个字段了，password字段改成了
authentication_string


	所以更改语句替换为update mysql.user set authentication_string=password('root') where user='root' ;即可
	
	然后删除之前在my.cnf中添加的那句话，保存后重启mysql即可