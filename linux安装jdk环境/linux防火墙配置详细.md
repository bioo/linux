# linux防火墙配置详细
在了解iptables的详细原理之前，我们先来看下如何使用iptables,以终为始，有可能会让你对iptables了解更深

所以接下来我们以配置一个生产环境下的iptables为例来讲讲它的常用命令

 

第一步：清空当前的所有规则和计数
iptables -F  #清空所有的防火墙规则
iptables -X  #删除用户自定义的空链
iptables -Z #清空计数
 

第二步：配置允许ssh端口连接

复制代码

iptables -A INPUT -s 192.168.1.0/24 -p tcp --dport 22 -j ACCEPT  #22为你的ssh端口， -s 192.168.1.0/24表示允许这个网段的机器来连接，其它网段的ip地址是登陆不了你的机器的。 -j ACCEPT表示接受这样的请求

复制代码

 


第三步：允许本地回环地址可以正常使用
iptables -A INPUT -i lo -j ACCEPT  #本地圆环地址就是那个127.0.0.1，是本机上使用的,它进与出都设置为允许
iptables -A OUTPUT -o lo -j ACCEPT
 

第四步：设置默认的规则
(由于在生产上，我们设置默认的入与转发都不允许，出的允许)

iptables -P INPUT DROP #配置默认的不让进
iptables -P FORWARD DROP #默认的不允许转发
iptables -P OUTPUT ACCEPT #默认的可以出去
 

 

第五步：配置白名单
iptables -A INPUT -p all -s 192.168.1.0/24 -j ACCEPT  #允许机房内网机器可以访问
iptables -A INPUT -p all -s 192.168.140.0/24 -j ACCEPT  #允许机房内网机器可以访问
iptables -A INPUT -p tcp -s 183.121.3.7 --dport 3380 -j ACCEPT #允许183.121.3.7访问本机的3380端口
 

 

第六步：开启相应的服务端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT #开启80端口，因为web对外都是这个端口
iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT #允许被ping
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT #已经建立的连接得让它进来
 

第七步：保存规则到配置文件中
因为刚刚的所有的规则都还是在内存中的，如果重启机器或者执行service iptables restart都会让其它失效，所以我们要把它保存在文件中，让它重启的时候能够被加载到。

复制代码
[root@zejin238 ~]# cp /etc/sysconfig/iptables /etc/sysconfig/iptables.bak #任何改动之前先备份，请保持这一优秀的习惯
[root@zejin238 ~]# iptables-save > /etc/sysconfig/iptables 
[root@zejin238 ~]# cat /etc/sysconfig/iptables
# Generated by iptables-save v1.4.7 on Wed Sep 28 18:06:07 2016
*filter
:INPUT DROP [8:632]  
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [4:416]
-A INPUT -s 192.168.1.0/24 -p tcp -m tcp --dport 22 -j ACCEPT 
-A INPUT -i lo -j ACCEPT 
-A INPUT -s 192.168.1.0/24 -j ACCEPT 
-A INPUT -s 192.168.140.0/24 -j ACCEPT 
-A INPUT -s 183.121.3.7/32 -p tcp -m tcp --dport 3380 -j ACCEPT 
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT 
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT 
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT 
-A OUTPUT -o lo -j ACCEPT 
COMMIT
# Completed on Wed Sep 28 18:06:07 2016
复制代码
 

至次，我们完成了生产环境iptables的配置

 

iptables文件说明
前面四行

复制代码
*filter #代表接下来的配置都是在filter表上的。我们默认的配置都在filter表上的，当然还有其它表，如raw,mangle,nat

:INPUT DROP [8:632]   #代表filter表上默认的input chain为drop ，对应上面的命令iptables -P INPUT DROP，中括号里面的两个数字代表的是这条链上已经接受到的包的数量及字节数量[包的数量:包的总字节数]
:FORWARD DROP [0:0]   #代表filter表上默认的forward chain为drop ，对应上面的命令iptables -P FORWARD DROP，中括号里面的两个数字代表的是这条链上已经接受到的包的数量及字节数量[包的数量:包的总字节数]
:OUTPUT ACCEPT [4:416] #代表filter表上默认的forward chain为drop ，对应上面的命令iptables -P OUTPUT ACCEPT，中括号里面的两个数字代表的是这条链上已经接受到的包的数量及字节数量[包的数量:包的总字节数]
复制代码
 

接下来的部分到commit结束前，都是我们刚刚执行的配置命令

复制代码
-A INPUT -s 192.168.1.0/24 -p tcp -m tcp --dport 22 -j ACCEPT 
-A INPUT -i lo -j ACCEPT 
-A INPUT -s 192.168.1.0/24 -j ACCEPT 
-A INPUT -s 192.168.140.0/24 -j ACCEPT 
-A INPUT -s 183.121.3.7/32 -p tcp -m tcp --dport 3380 -j ACCEPT 
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT 
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT 
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT 
-A OUTPUT -o lo -j ACCEPT 
复制代码
input chain与output chain分开，其它的顺序与我们配置时的顺序一致。

 

 

iptables优化
当服务器收到一条请求时，它会把iptables从上往下，一条条匹配定制的规则，那么假如机器收到一个正常的web请求，要走80端口，它需要先去检验前面5条规则，发现都不符合，直到第六条满足条件，那么这样的话防火墙的工作效率就低了很多。

所以我们优化的思路是：请求最频繁的放在最上面，请求频率较小的放在最后面。

我们此时可以直接去修改文件：vi /etc/sysconfig/iptables

调整后顺序大概为：

复制代码
# Generated by iptables-save v1.4.7 on Wed Sep 28 18:06:07 2016
*filter
:INPUT DROP [8:632]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [4:416]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -s 192.168.135.0/24 -j ACCEPT
-A INPUT -s 192.168.140.0/24 -j ACCEPT
-A INPUT -s 183.121.3.7/32 -p tcp -m tcp --dport 3380 -j ACCEPT
-A INPUT -s 192.168.1.0/24 -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A OUTPUT -o lo -j ACCEPT
COMMIT
# Completed on Wed Sep 28 18:06:07 2016
复制代码
因为web服务器，肯定是80端口访问最频繁，那为什么不是放第一条呢？

在这里，我们把已经有状态的联接放在第一条是因为在请求连接中，有很多都是在建立在已经联接的基础上通信的，譬如有可能你第一次连接的3380端口，状态是new，但在后续的联接都为ESTABLISHED，第一条就可以匹配到了。

 

 

理解iptables格式
我们在上面的iptables文件中，最开始接触会发现格式蛮奇怪的，不知道什么回事

其实iptables就是定义一些规则，满足相应的规则则进行ACCEPT、DROP、REJECT、DNAT、SNAT......

那么怎么定义匹配规则：

通用匹配

  -s 指定源地址

  -d 指定目标地址

  -p 指定协议

  -i 指定数据报文流入接口

  -o 指定数据报文流出接口

 

扩展匹配

  指定-m选项，表示用什么模块来匹配，如：

  -m state --state

           NEW,ESTABLISHED,RELATED  表示用state模块来匹配当前连接状态为这三种状态的连接

  -m iprange

       --src-range 用iprange模块匹配来源的ip地址范围

       --dst-range 用iprange模块匹配目的的ip地址范围

   -m multiport

        --source-ports  用multiport模块来匹配来源的端口范围

        --destination-ports 用multiport模块来匹配目的的端口范围

我们看如下一条例子来解读：

-A INPUT -p tcp -m iprange --src-range 121.21.30.36-121.21.30.100 -m multiport --destination-ports 3326,3327,3328 -m state --state NEW -j ACCEPT

这表示来自121.21.30.36-121.21.30.100这个地址范围的，请求端口为3326,3327,3328并且是新建的连接，都给予放行。

 

模块有很多，参数有很多，有些没看过怎么办？

man iptables

 

那iptables还有其它表，内部是怎么流转的，请看 Linux iptables原理--数据包流向 。