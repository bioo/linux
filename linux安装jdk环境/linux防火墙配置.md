# linux防火墙配置

centos7 开放3306端口并可以远程访问
2018年01月04日 12:02:06 yang洋PHPer 阅读数：3071
版权声明：本文为博主原创文章，未经博主允许不得转载。	https://blog.csdn.net/qq_34341290/article/details/78969485
开启远程访问：


GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION; 允许任何ip以root用户登录

flush privileges;立即生效


CentOS 7.0默认使用的是firewall作为防火墙，这里改为iptables防火墙。
1、关闭firewall：
```
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl mask firewalld.service
```

2、安装iptables防火墙
```
yum install iptables-services -y
```

3.启动设置防火墙

```
# systemctl enable iptables
# systemctl start iptables
```

4.查看防火墙状态

```
systemctl status iptables
```

5编辑防火墙，增加端口

```
vi /etc/sysconfig/iptables #编辑防火墙配置文件
```
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
```
:wq! #保存退出

3.重启配置，重启系统

```
systemctl restart iptables.service #重启防火墙使配置生效
systemctl enable iptables.service #设置防火墙开机启动
```