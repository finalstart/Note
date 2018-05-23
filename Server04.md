一、httpd基础网络服务（rhel5自带的httpd）
1，软件包
httpd 
2，相应文件
服务目录		/etc/httpd/
主配置文件		/etc/httpd/conf/httpd.conf
网站根目录		/var/www/html/
缺省没主页时访问页面	/var/www/error/noindex.html
访问日志		/var/log/httpd/access_log
错误日志		/var/log/httpd/error_log
系统服务脚本		/etc/init.d/httpd
		
3，默认端口
	TCP 80,
4，配置文件结构
全局设置：配置参数   值
目录设置：
    <Directory 目录> .. .. </Directory>
访问位置设置：
    <Location URL> .. .. </Location>
虚拟主机设置
    <VirtualHost 监听地址> .. .. </VirtualHost>
5，常用的全局设置参数
ServerName	本站点的FQDN名称
DocumentRoot	网页文档的根目录
DirectoryIndex	默认索引页/首页文件
ErrorLog		错误日志文件的位置
CustomLog	访问日志文件的位置
Listen		监听服务的IP地址、端口号
ServerRoot	服务目录
Timeout		网络连接超时，默认 300 秒
KeepAlive		是否保持连接，可选On或Off
MaxKeepAliveRequests	每次连接最多处理的请求数
KeepAliveTimeout		保持连接的超时时限
Include			可包含其他子配置文件

二、httpd访问控制
1，客户机地址限制
Order 配置项，定义控制顺序
	allow,deny  先允许后拒绝，默认拒绝所有
	deny,allow  先拒绝后允许，默认允许所有
Allow/Deny from 配置项，设置权限
	Deny  from  地址1  地址2  .. ..
	Allow  from  地址1  地址2  .. ..
2，用户授权
AuthName	认证领域名称，用于弹窗提示
AuthType	认证类型，一般使用basic
AuthUserFile	用户数据文件的路径
Require		指定授权用户或组
3，httpd工作模式
prefork	预创建、非线程模式，2.2.x 系列默认
worker	多线程，多处理模式
# httpd -l 查看工作模式
4，目录别名
Alias  /error/  "/var/www/error/"
5，Options选项说明：这个写在Directory里面
Options 
 Indexes      当站点里面没有默认主页时，把站点里面的内容用列表显示出来  FollowSymLinks 允许站点里面存在链接 

三、AWStats日志分析
1，PV(访问量)：即Page View, 页面浏览量或点击量，用户每次刷新即被计算一次。 
   UV(独立访客)：即Unique Visitor,访问您网站的一台电脑客户端为一个访客。00:00-24:00内相同的客户端只被计算一次。
   IP(独立IP)：即Internet Protocol,指独立IP数。00:00-24:00内相同IP地址之被计算一次。 



[root@ser1 ~]# grep HTTPD /etc/sysconfig/httpd
#HTTPD=/usr/sbin/httpd.worker      //以线程方式启动
默认以进程方式启动














httpd-2.2.x(prefork)
httpd-2.4.x(event)

PHP官方推荐httpd使用prefork(php能更稳定地运行),而不是线程化的worker和event,httpd-2.4.x默认使用线程化的event作为mpm.

Linux上很多PECL库都是非线程安全的,libphp5.so在线程化的httpd(event/worker)中运行可能会出现一些问题,为了保持兼容性和稳定性,PHP一般还是使用httpd-2.2.x(prefork)这个分支. 

针对主机服务商和开发人员，Apache 2.4提供了很多性能方面的提升，包括支持更大流量、更好地支持云计、利用更少的内存处理更多的并发等。
除此之外，新版Apache的提升还包括性能提升、内存利用、异步I/O的支持、动态反向代理设置、与时间驱动的Web服务器相当或更好的性能、更强大的处理资源分配能力，更便捷的缓存支持以及可定制的高速服务器和代理等。


----------

		              	    RHEL5.9_A HTTP Server
-------Server1（VM1）---------（VM1） 
      192.168.10.254                RHEL5.9_B Client

实验要求：
	主机名设为：www.tarena.com  192.168.10.10
	默认首页包括：index.html、index.php
	确认默认httpd是否支持php	
	
一、搭建DNS（host与DNS任选一个即可）
1）host文件
192.168.10.10	www.tarena.com	www
2）构建DNS（tarena.com）服务器		
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
dns1   IN      A       192.168.10.10
www    IN      A       192.168.10.10
4、启动服务
[root@ser1 named]# service named restart
[root@ser1 named]# chkconfig named on

二、搭建HTTPD
1、安装软件包
[root@ser1 ~]# yum -y install httpd 
2、修改主配置文件
[root@ser1 ~]# vim /etc/httpd/conf/httpd.conf 
...
265 ServerName www.tarena.com:80
...
391 DirectoryIndex index.html index.php
...
[root@ser1 ~]# cat /var/www/html/index.php
<?php
        phpinfo();
?>
3、启动服务
[root@ser1 ~]# service httpd restart
[root@ser1 ~]# chkconfig httpd on

测试：
http://www.tarena.com
[root@ser1 ~]# cat /var/www/html/index.html
<h1>Test Page!!!</h1>
http://www.tarena.com

实验要求：
	只允许192.168.10.20访问www.tarena.com
	允许访问所有客户端访问www.tarena.com/authdir/index.html
1、修改主配置文件
[root@ser1 ~]# mkdir /var/www/html/authdir
[root@ser1 ~]# cat /var/www/html/authdir/index.html
<h1>This is a Test BBS Page!!!</h1>
[root@ser1 ~]# vim /etc/httpd/conf/httpd.conf 
...
306 <Directory "/var/www/html">
...
332     Order allow,deny
333 #    Allow from all
334     Allow from 192.168.10.20
...
337 <Directory "/var/www/html/authdir">
338         Order allow,deny
339         Allow from all
340 </Directory>
2、启动服务
[root@ser1 ~]# service httpd restart
3、在不同客户端测试
[root@ser1 ~]# tail /var/log/httpd/error_log 
...
[Mon Nov 25 11:54:29 2013] [error] [client 192.168.10.100] client denied by server configuration: /var/www/html/

实验要求：
	客户端访问/var/www/html/authdir/需要输入用户名密码验证

1、修改主配置文件
[root@ser1 ~]# vim /etc/httpd/conf/httpd.conf 
...
337 <Directory "/var/www/html/authdir">
338         Order allow,deny
339         Allow from all
340         AuthName "Please Input Password!!"
341         AuthType Basic
342         AuthUserFile "/etc/httpd/auth.ulist"
343         Require valid-user
344 </Directory>
...
2、创建账户密码
[root@ser1 ~]# htpasswd -c /etc/httpd/auth.ulist admin
New password: 
Re-type new password: 
Adding password for user admin
3、启动服务测试
[root@ser1 ~]# service httpd restart
http://www.tarena.com/authdir

实验要求：
	客户端访问http://www.tarena.com/sina时可以访问/var/www/html/sina.com/bbs下的网页
1、创建测试站点
[root@ser1 ~]# mkdir -p /var/www/html/sina.com/bbs/
[root@ser1 ~]# cat /var/www/html/sina.com/bbs/index.html
<h1>This is bbs for Sina!!!</h1>
2、修改主配置文件
[root@ser1 ~]# vim /etc/httpd/conf/httpd.conf
...
 548 Alias /sina "/var/www/html/sina.com/bbs"
3、启动服务测试
[root@ser1 ~]# service httpd restart
http://www.tarena.com/sina


实验要求：
	部署Awstats统计Http访问日志
1、安装软件（软件在/usr/src下）
[root@ser1 ~]# cd /usr/src/
[root@ser1 src]# tar -zxf awstats-7.1.tar.gz 
[root@ser1 src]# mv awstats-7.1 /usr/local/awstats
2、为站点建立配置文件
[root@ser1 src]# cd /usr/local/awstats/tools/
[root@ser1 tools]# ./awstats_configure.pl
...
Config file path ('none' to skip web server setup):
> /etc/httpd/conf/httpd.conf
...
-----> Need to create a new config file ?
Do you want me to build a new AWStats config/profile
file (required if first install) [y/N] ? y   
...
Your web site, virtual server or profile name:
> www.tarena.com
...
Default: /etc/awstats
Directory path to store config file(s) (Enter for default):
> 
...
/usr/local/awstats/tools/awstats_updateall.pl now
Press ENTER to continue... 
...
Press ENTER to finish...
3、指定统计的目标日志文件
[root@ser1 tools]# vim /etc/awstats/awstats.www.tarena.com.conf 
...
  51 LogFile="/var/log/httpd/access_log"
[root@ser1 tools]# mkdir /var/lib/awstats
4、将日志文件导入Awstats
[root@ser1 tools]# ./awstats_updateall.pl now
[root@ser1 tools]# crontab -e
*/5 * * * *　/usr/local/awstats/tools/awstats_updateall.pl now
[root@ser1 tools]# service crond restart
[root@ser1 tools]# chkconfig crond on

验证：
http://www.tarena.com/awstats/awstats.pl?config=www.tarena.com

[root@ser1 tools]# cat /var/www/html/awstats.html
<html>
<head><meta http-equiv=refresh content="0; url=http://www.tarena.com/awstats/awstats.pl?config=www.tarena.com">
</head>
<body>
</body>
</html>
验证：
http://www.tarena.com/awstats.html


