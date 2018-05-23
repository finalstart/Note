mysql主从同步
master                 slave
192.168.1.1            192.168.1.100


一、公共配置
1 service mysqld  start
2 ping通
3 service  iptables  stop
  setenforce  0


二、配置Master
2.1
在master服务器上授权 用户可以从 slave 服务器上连接自己，且有拷贝数据的权限
grant  replication  slave  on  *.* to  jim@"slave数据库服务器的ip" identified by "密码"；

2.2 编辑自己的/etc/my.cnf
[mysqld]
log-bin
server-id=1

2.3
service mysqld restart


三、配置slave
3.1  service mysqld start
3.2  从数据库服务器上一定要有主数据库服务器上的库和表 且表结构要     与主数据库服务器的表结构一致
3.3  测试使用授权用户是否能登陆主数据库服务器
     mysql  -hmaster主机的ip地址  -umaster主机的授权用户 -p授权用     户的密码

3.3  设置自己是哪台数据库服务器的slave服务器。
vim /etc/my.cnf
[mysqld]
server-id=100
master-host=master主机的ip地址
master-user=master主机的授权用户
master-password=授权用户的密码
log-bin  可选项

service  mysqld  restart

四、测试slave能否与master进行数据同步。



五、mysql主从同步的工作过程
mysql> show master status;

mysql> show slave status\G;
Slave_IO_Running:  Yes
Slave_SQL_Running: Yes


IO  连接master服务器，并把master服务器binlog文件里的sql语句，拷贝      到本机的relaybinlog文件里  

IO  进程连接不上主数据库服务器时会处于NO状态
         

SQL 执行本机relaybinlog文件里的sql语句。

SQL 本机没有主数据库服务器上的库和表，或表结构有master不一致时，会     处于NO状态

slave：
mysql> slave  stop;
mysql> change  master to  master_log_file='mysqld-bin.000001',
master_log_pos=1348;

mysql>slave start;

mysql>  change  master  to  选项=值,选项=值；
选项
master_host=‘master主机ip’
master_user=‘授权用户’
master_password=‘密码’
master_log_file=‘binlog日志名’
master_log_pos=‘节点数’



slave主机的相关文件
master.info      记录master服务器的信息        
mysqld-relay-bin.index    记录从数据库服务器的binlog文件列表
mysqld-relay-bin.000002   从数据库服务器上的binlog文件
relay-log.info   记录binlog文件信息


mysql主从同步结构
1、 一主一从
192.168.1.1  （主）      192.168.1.200 （从）

2、一主多从

              201
192.168.1.1   210
              218

3主从从 （配置）

主                 从                从
192.168.1.1    192.168.1.100    192.168.1.178


4 互为主从  （课后作业）

主                  从
192.168.1.10        192.168.1.213
从                  主


课堂练习
配置主从从mysql主从同步结构。
主              从              从
192.168.1.1     192.168.1.10    192.168.1.30    
  
要求：
1、
192.168.1.10 从数据库服务器备份192.168.1.1数据库服务器上的所有数据

2、
192.168.1.30 从数据库服务器备份192.168.1.10数据库服务器上除mysql数据库之外的所有库的数据。

问：
服务器192.168.1.30会不会有服务器192.168.1.1上的数据？

从数据库服务器本机的SQL进程执行的sql语句，是否会写进本机的binlog日志文件里？

           mysql -h192.168.1.254  -u用户名 -p密码
           cliet  
             |
       |mysql-proxy|  192.168.1.254
             |
    --------------------
         |      |    
  （写）主     从(读)
       118     217


118+217
1 service myqld start
2 库、表、表结构要一致
118 create  database  db10;
    create table db10.a(id int);
    insert into db10a values(118);

217
    create  database  db10;
    create table db10.a(id int);
    insert into db10a values(217);
3 在负责读写操作的2台数据库服务器上授权允许从mysql代理服务器连接自己

118
    grant all on *.*  to  jim@"%"  identified by "123";
217

        
mysql-proxy-server:
0 测试使用授权用户能否在本机登陆后端提供数据库服务器主机
mysql -h192.168.1.217 -ujim -p123
mysql -h192.168.1.118 -ujim -p123

1
service mysqld stop
chkconfig --level 35 mysqld off

2 安装提供代理服务的软件


chmod +x share/doc/mysql-proxy/rw-splitting.lua


3 启动代理服务 
cd  安装目录/bin
./mysql-proxy  -P   192.168.1.254:3306  -r 192.168.1.217:3306
-b 192.168.1.118:3306  -s 安装目录/share/doc/mysql-proxy/rw-splitting  &


-P    代理服务器ip：端口
-r    读服务器ip：端口
-b    写服务器ip：端口
-s    区分用户读写操作的脚本文件
&     后台运行


测试代理服务器？
mysql  -h代理服务器  -ujim   -p123

验证
并发访问量达到多少时区分读写请求？
并发访问量小于阀值时把请求发给那台主机？
并发访问量缩减到小于阀值后是否依然区分读写请求？















