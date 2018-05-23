设置数据库管理员的初始密码
mysqladmin -uroot -hlocalhost password  "123456"

有密码时登陆的格式
mysql -uroot -hlocalhost -p123456


重置数据库管理员密码

mysqladmin  -h数据库服务器ip/主机名 -u数据库管理员名  -p  password    "888"

管理员用新密码登陆
mysql -uroot -hlocalhost -p888 



恢复数据库管理员密码

vim /etc/my.cnf
[mysqld]
skip-grant-table

service mysql restart

mysql
mysql> update mysql.user set password=password("123") where user="root" and host="localhost";

mysql> flush privileges;


mysql -uroot -hlocalhost -p123


用户授权
(默认情况下：只有数据库管理员从数据库服务器本机登陆才有授权权限)

mysql> show grants;

授权命令格式：

grant  权限列表  on  数据库名  to  用户名@"客户端地址"  
identified   by  "密码"   with grant option; 

权限列表表示：
all  所有权限
select,delete,update  指定有某种权限
select,update(name,sex,age)  指定有某种权限

数据库名表示方式
*.*  服务器上的所有库所有表
数据库名.*  某个库里所有表
数据库名.表名   某个库下的某个表

用户名表示方式
管理员授权时自定义的用户名(mysql.user) 与系统账号无关(/etc/passwd)
要有标识性


客户端地址：
所有主机     %  
某个Ip地址  1.1.1.1
某个网段    192.168.1.%
主机名      pc1.tarena.com  (数据服务器能够解析主机名)
区域        %.tarena.com

identified   by  "密码"   
设置授权用户连接数据库服务器时使用的密码
是可选项 
若授权时不写此选项用户登录时没有密码

with grant option
授权用户是否有授权权限
可选项、 
若授权时不写此选项用户登录时没有授权权限


如何查看当前数据库服务器上有哪些授权用户？权限是什么样的？

mysql> show grants for root@"192.168.1.2";


练习
授权数据库管理员root账户可以从地址是192.168.1.2主机连接数据库服务器192.168.1.1  连接密码是666 对所有库所有表有完全权限 且有授权的权限。

允许使用webuser账户从网络中的所有主机访问数据库服务器，只对数据库服务器上的webdb有完全权限。密码时webuser88

grant all  on *.*  to root@"192.168.1.2" identified by "666" with grant option;

撤销授权
revoke 权限列表  on 数据库名  from  用户名@"客户端地址"；

 
http://www.51cto/bbs/xxx.php


数据库管理员密码的设置（初始密码 重置密码 忘记密码）
用户授权（查看权限 查看授权用户  授权）
撤销权限 （revoke）
图形管理工具 phpmyadmin





















