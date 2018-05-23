Kisckstart无人值守安装
DHCP
TFTP
NFS
FTP/HTTP


一、DNS域名服务器基础
1，概念
	DNS Domain Name System 域名解析系统
2，dns作用
	正向解析：将域名转换成对应的ip地址
	反向解析：将ip地址转换成对应的域名
3，域名解析过程
	 一次递归多次迭代 
4，使用端口  tcp/udp 53
5，dns服务器类型
	缓存服务器
	主域名服务器
	从域名服务器
6，软件包
	bind			服务器软件包
	bind-chroot		切换路径包
	caching-nameserver	模板文件
7，主配置文件
	/var/named/chroot/etc/named.conf
8，区域文件目录
	/var/named/chroot/var/named 
9，type类型
	hint	根区域 
	master	主区域
	slave	从区域
	forward	转发区域
10，检测命令
	named-checkconf
	named-checkzone  
11，区域文件的内容：
	@	代表当前域
	NS 	域名服务器记录
	A   	将主机名--->ip
	MX 	邮件交换记录
	PTR 	ip--->主机名
	CNAME 	别名
12，DNS负载均衡
	同一个域名对应到多个ip
13，泛域名解析
	找不到精确对应的A记录时，以*条目匹配
$GENERATE       1-200   station$        IN A            192.168.0.$
$GENERATE       1-200   $       IN PTR          station$.example.com.

$GENERATE 是函数 
1-200 是要循环的变量
station$是主机名 
192.168.1.$是对应的IP地址

二、构建主/从DNS服务器
1，主域名服务器
 	allow-transfer { 10.0.0.20； }；
2，从域名服务器
	type slave;
	file "slaves/xxx";
	masters { 10.0.0.10 ; };
2，服务的启动
	/etc/init.d/named restart
3，测试
	nslookup
	dig
	host -t NS
	 	SOA
	 	MX
           -a ns1.tarena.com
           -l  tarena.com. 10.0.0.10


----------
一、DNS子域授权
dns是分层负责解析的，子域授权就是在一个域内新建几个子域，然后客户端的dns指向父域的地址，能够解析到父域管辖内子域的记录，子域的客户端把DNS指向子域能够解析父域的地址。减轻父域的负担

二、分离解析（DNS视图）
根据不同客户端的来源地址在访问相同域名时，可以给出不同解析结果

三、缓存DNS
1，forwarders { ip; };定义在主配置文件中

## DNS主从实验搭建
		              	    RHEL5.9_A DNS Master
-------Server1（VM1）---------（VM1） 
    192.168.10.254                  RHEL5.9_B DNS Slave

实验要求：
	搭建主DNS服务器
	tarena.com
		mail.tarena.com	192.168.10.20
		www.tarena.com	192.168.10.10
		ftp.tarena.com	192.168.10.10
1、安装软件包
[root@ser1 ~]# yum -y install bind bind-chroot caching-nameserver
2、修改主配置文件
[root@ser1 ~]# cd /var/named/chroot/etc/
[root@ser1 etc]# cp -p named.caching-nameserver.conf named.conf
[root@ser1 etc]# vim named.conf
...
 15         listen-on port 53 { 192.168.10.10; };
...
 27         allow-query     { any; };
 28         allow-query-cache { any; };
...
 37         match-clients      { any; };
 38         match-destinations { any; };
[root@ser1 etc]# vim named.rfc1912.zones 
...
 51 zone "tarena.com" IN {
 52         type master;
 53         file "t.zheng";
 54 };
 55 
 56 zone "10.168.192.in-addr.arpa" IN {
 57         type master;
 58         file "t.fan";
 59 };
[root@ser1 etc]# named-checkconf named.conf 
3、修改数据库文件
[root@ser1 etc]# cd /var/named/chroot/var/named/
[root@ser1 named]# cp -p named.local t.zheng 
[root@ser1 named]# cat t.zheng 
$TTL    86400
@       IN      SOA     tarena.com. root.tarena.com.  (
                                      2013112101 ; Serial
                                      28800      ; Refresh
                                      14400      ; Retry
                                      3600000    ; Expire
                                      86400 )    ; Minimum
            IN      NS      dns1.tarena.com.
dns1    IN      A       192.168.10.10
mail     IN      A       192.168.10.20
www    IN      A       192.168.10.10
ftp       IN      CNAME   www
[root@ser1 named]# cp -p t.zheng t.fan 
[root@ser1 named]# cat t.fan 
$TTL    86400
@       IN      SOA     tarena.com. root.tarena.com.  (
                                      2013112101 ; Serial
                                      28800      ; Refresh
                                      14400      ; Retry
                                      3600000    ; Expire
                                      86400 )    ; Minimum
        IN      NS      dns1.tarena.com.
10      IN      PTR     dns1.tarena.com.
20      IN      PTR     mail.tarena.com.
10      IN      PTR     www.tarena.com.
10      IN      PTR     ftp.tarena.com.
4、启动服务
[root@ser1 named]# service named restart
[root@ser1 named]# chkconfig named on

客户端测试：
先设置客户端的dns地址，在测试

实验要求：
	搭建从DNS服务器
	tarena.com
		mail.tarena.com	192.168.10.20
		www.tarena.com	192.168.10.10
		ftp.tarena.com	192.168.10.10
1、安装软件包
[root@ser2 ~]# yum -y install bind bind-chroot caching-nameserver
2、修改主配置文件
[root@ser2 ~]# cd /var/named/chroot/etc/
[root@ser2 etc]# cp -p named.caching-nameserver.conf named.conf
...
 15         listen-on port 53 { 192.168.10.20; };
...
 27         allow-query     { any; };
 28         allow-query-cache { any; };
...
 37         match-clients      { any; };
 38         match-destinations { any; };
[root@ser2 etc]# vim named.rfc1912.zones 
...
 51 zone "tarena.com" IN {
 52         type slave;
 53         file "slaves/t.zheng";
 54         masters { 192.168.10.10; };
 55 };
 56 
 57 zone "10.168.192.in-addr.arpa" IN {
 58         type slave;
 59         file "slaves/t.fan";
 60         masters { 192.168.10.10; };
 61 };
[root@ser2 etc]# named-checkconf named.conf 
3、启动服务并验证
[root@ser2 etc]# ls /var/named/chroot/var/named/slaves/
[root@ser2 etc]# service named restart
停止 named：                                               [确定]
启动 named：                                               [确定]
[root@ser2 etc]# ls /var/named/chroot/var/named/slaves/
t.fan  t.zheng


## DNS子域授权实验
		              	               RHEL5.9_A tarena.com
-------Server1（VM1）---------（VM1） 
          192.168.10.254                              RHEL5.9_B bj.tarena.com

实验要求：
	构建父DNS（tarena.com）服务器
	构建子DNS（bj.tarena.com）服务器
	在父DNS服务器上配置子域授权

构建父DNS（tarena.com）服务器		
1、安装软件包
[root@ser1 ~]# yum -y install bind bind-chroot caching-nameserver
2、修改主配置文件
[root@ser1 ~]# cd /var/named/chroot/etc/
[root@ser1 etc]# cp -p named.caching-nameserver.conf named.conf
[root@ser1 etc]# vim named.conf
...
 15         listen-on port 53 { 192.168.10.10; };
...
 27         allow-query     { any; };
 28         allow-query-cache { any; };
...
 37         match-clients      { any; };
 38         match-destinations { any; };
[root@ser1 etc]# vim named.rfc1912.zones 
...
 51 zone "tarena.com" IN {
 52         type master;
 53         file "tarena.com.zone";
 54 };
[root@ser1 etc]# named-checkconf named.conf 
3、修改数据库文件
[root@ser1 etc]# cd /var/named/chroot/var/named/
[root@ser1 named]# cp -p named.local tarena.com.zone 
[root@ser1 named]# cat tarena.com.zone 
$TTL    86400
@       IN      SOA     tarena.com. root.tarena.com.  (
                                      2013112101 ; Serial
                                      28800      ; Refresh
                                      14400      ; Retry
                                      3600000    ; Expire
                                      86400 )    ; Minimum
            IN      NS    dns1.tarena.com.
bj.tarena.com.	IN	NS	dns1.bj.tarena.com.
dns1.bj.tarena.com.	IN	A	192.168.10.20
dns1    IN      A       192.168.10.10
www    IN      A       192.168.10.10
4、启动服务
[root@ser1 named]# service named restart
[root@ser1 named]# chkconfig named on

构建子DNS（bj.tarena.com）服务器
1、安装软件包
[root@ser2 ~]# yum -y install bind bind-chroot caching-nameserver
2、修改主配置文件
[root@ser2 ~]# cd /var/named/chroot/etc/
[root@ser2 etc]# cp -p named.caching-nameserver.conf named.conf
...
 15         listen-on port 53 { 192.168.10.20; };
...
21          forwarders { 192.168.10.10; };
...
 27         allow-query     { any; };
 28         allow-query-cache { any; };
...
 37         match-clients      { any; };
 38         match-destinations { any; };
[root@ser2 etc]# vim named.rfc1912.zones 
...
 51 zone "bj.tarena.com" IN {
 52         type master;
 53         file "bj.tarena.com.zone";
 54 };
[root@ser2 etc]# named-checkconf named.conf 
3、修改数据库文件
[root@ser2 etc]# cd /var/named/chroot/var/named/
[root@ser2 named]# cp -p named.local bj.tarena.com.zone 
[root@ser2 named]# cat bj.tarena.com.zone 
$TTL    86400
@       IN      SOA     bj.tarena.com. bj.root.tarena.com.  (
                                      2013112101 ; Serial
                                      28800      ; Refresh
                                      14400      ; Retry
                                      3600000    ; Expire
                                      86400 )    ; Minimum
            IN      NS    dns1.bj.tarena.com.
dns1    IN	     A       192.168.10.20
www    IN      A       192.168.10.100
4、启动服务并验证
[root@ser2 etc]# service named restart
[root@ser2 etc]# chkconfig named on
5、测试
[root@ser2 etc]# host www.bj.tarena.com 192.168.10.10
[root@ser2 etc]# host www.tarena.com 192.168.10.20

## DNS分离解析
		              	    RHEL5.9_A DNS Server
-------Server1（VM1）---------（VM1） 
      192.168.10.254                RHEL5.9_B Client

实验要求：
	192.168.10.20解析www.tarena.com 结果192.168.10.100
	192.168.10.21解析www.tarena.com 结果192.168.10.101
	其他主机解析www.tarena.com 结果192.168.10.102
构建DNS（tarena.com）服务器		
1、安装软件包
[root@ser1 ~]# yum -y install bind bind-chroot caching-nameserver
2、修改主配置文件
[root@ser1 ~]# cd /var/named/chroot/etc/
[root@ser1 etc]# cp -p named.caching-nameserver.conf named.conf
[root@ser1 etc]# vim named.conf
...
 14 acl lt { 192.168.10.20; };
 15 acl yd { 192.168.10.21; };
...
 17         listen-on port 53 { 192.168.10.10; };
...
 29         allow-query     { any; };
 30         allow-query-cache { any; };
...
 38 view lt_resolver {
 39         match-clients      { lt; };
 40         match-destinations { any; };
 41         recursion yes;
 42         include "/etc/named.rfc1912.zones";
 43 };
 44 view yd_resolver {
 45         match-clients      { yd; };
 46         match-destinations { any; };
 47         recursion yes;
 48         include "/etc/named.yd.zones";
 49 };
 50 view other_resolver {
 51         match-clients      { any; };
 52         match-destinations { any; };
 53         recursion yes;
 54         include "/etc/named.other.zones";
 55 };
[root@ser1 etc]# vim named.rfc1912.zones 
...
  51 zone "tarena.com" IN {
 52         type master;
 53         file "lt.zone";
 54 };
[root@ser1 etc]# vim named.yd.zones 
...
 51 zone "tarena.com" IN {
 52         type master;
 53         file "yd.zone";
 54 };
[root@ser1 etc]# vim named.other.zones 
...
 51 zone "tarena.com" IN {
 52         type master;
 53         file "other.zone";
 54 };
[root@ser1 etc]# ln -s /var/named/chroot/etc/named.other.zones  /etc/named.other.zones
[root@ser1 etc]# ln -s /var/named/chroot/etc/named.yd.zones  /etc/named.yd.zones
[root@ser1 etc]# named-checkconf named.conf 
3、修改数据库文件
[root@ser1 etc]# cd /var/named/chroot/var/named/
[root@ser1 named]# cp -p named.local lt.zone 
[root@ser1 named]# cat lt.zone 
$TTL    86400
@       IN      SOA     tarena.com. root.tarena.com.  (
                                      2013112101 ; Serial
                                      28800      ; Refresh
                                      14400      ; Retry
                                      3600000    ; Expire
                                      86400 )    ; Minimum
            IN      NS    dns1.tarena.com.
dns1   IN      A       192.168.10.10
www    IN      A       192.168.10.100
[root@ser1 named]# cp -p lt.zone yd.zone 
[root@ser1 named]# cat yd.zone 
$TTL    86400
@               IN SOA  tarena.com.      root.tarena.com. (
                                      2013122101      ; serial 
                                      3H              ; refresh
                                      15M             ; retry
                                      1W              ; expiry
                                      1D )            ; minimum
        IN      NS      dns1.tarena.com.
dns1    IN      A       192.168.10.10
www     IN      A       192.168.10.101
[root@ser1 named]# cp -p lt.zone other.zone 
[root@ser1 named]# cat other.zone 
$TTL    86400
@               IN SOA  tarena.com.      root.tarena.com. (
                                     2013122101      ; serial                                           3H              ; refresh
                                     15M             ; retry
                                     1W              ; expiry
                                     1D )            ; minimum
        IN      NS      dns1.tarena.com.
dns1    IN      A       192.168.10.10
www     IN      A       192.168.10.102
4、启动服务
[root@ser1 named]# service named restart
[root@ser1 named]# chkconfig named on

测试：
[root@ser1 ~]# ifconfig eth0 192.168.10.20
[root@ser1 ~]# host www.tarena.com 192.168.10.10
Using domain server:
Name: 192.168.10.10
Address: 192.168.10.10#53
Aliases: 

www.tarena.com has address 192.168.10.100
[root@ser1 ~]# ifconfig eth0 192.168.10.21
[root@ser1 ~]# host www.tarena.com 192.168.10.10
Using domain server:
Name: 192.168.10.10
Address: 192.168.10.10#53
Aliases: 

www.tarena.com has address 192.168.10.101
[root@ser1 ~]# ifconfig eth0 192.168.10.2
[root@ser1 ~]# host www.tarena.com 192.168.10.10
Using domain server:
Name: 192.168.10.10
Address: 192.168.10.10#53
Aliases: 

www.tarena.com has address 192.168.10.102
