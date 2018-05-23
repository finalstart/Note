mysql.*

rm -rf /var/lib/mysql/mysql/user.*

一、数据备份
1.1为什么要做数据备份？
1.2数据备份的备份方式

物理备份  cp   tar    tar+gzip

逻辑备份  备份创建库或表时的SQL语句
create database  db;
create table db.a(id int);
insert into db.a values(100);
insert into db.a values(200);

1.3 备份策略
完整备份     把数据库服务器上所有库所有表全部备份
差异备份     备份“自完整备份后” 产生的新数据
增量备份     备份 “自上一次备份后” 产生的新数据


完整备份+差异备份  （天 、周、月） 
100
1   完整备份    db1.a(10)   23:00              1.bak
2               db2.*      db2.*                2.bak
3               db3.*      db2 db3              3.bak
4               db4.*      db2 db3  db4
5               db5.*      db2 db3  db4 db5
6               db6.*      db2 db3  db4 db5 db6
7               db7.*      db2 db3  db4 db5 db6 db7  7.bak

1               10:00      down
 

完整备份+增量备份
1     完整备份    db10.b(10)     23:00     db10.*   1.bak
2     增量备份    db20.*         23:00     db20.*    2.bak
3     增量备份    db30.*         23:00     db30.*
4     增量备份    db40.*         23:00     db40.*
5     增量备份
6     增量备份
7     增量备份    db70.*         23:00      db70.*   7.bak

1     9:00 down 

备份工具
mysqldump   完整备份

格式
mysqldump  -h数据库服务器ip或主机名  -u登陆用户名  -p密码   数据库名   >   目录名/名字.sql  


数据库名表示方式
--all-databases   ：所有库所有表
数据库名          ：单个数据库
数据库名  表名    ：某个库下的某个表
-B 数据库名1  数据库名2  数据库名N     ：某几个库


二、数据恢复
mysql  -hip或主机名  -u登陆用户名  -p密码  数据库名  <  备份文件名

数据库名是可选项：当恢复数据时使用的备份文件里有建库使用库的SQL语句时，数据库名可以省略，反之必须写数据库名。


mysqldump -h192.168.1.1 -uroot -p123 web3 > web3-`date +%F`.sql

三、mysql日志类型（4）
默认情况下只有 “错误日志” 是开启的，其他3种日志 要想使用，要在/etc/my.cnf文件中开启。

1 错误日志 ：mysql服务启动或运行时产生的错误   
[mysqld_sefa]
log-error=/var/log/mysqld.log

2 查询日志 ：记录在数据库服务上执行的所有sql操作
[mysqld]
log

3 慢查询日志 : 记录超过指定时间（默认10秒）显示查询结果的SQL语句
[mysqld]
log-slow-queries
long_query_time=N

4 二进制日志(binlog日志) ：记录除查询之外的SQL 语句。
                            select  show   desc  
                            insert  delete  update  grant revoke

vim /etc/my.cnf
[mysqld]
log-bin[=/mysqllog/plj.log]
max_binlog_size=数字M  （默认大小500M）


mysqld-bin.000001 binlog日志文件 （日志文件编号从1开始）默认当当前的binlog日志大于500M时自动生成第二个binlog文件 名字为
mysqld-bin.000002

mysqld-bin.index  已有的binlog文件列表

mysqlbinlog    mysqld-bin.000001

如何生成新的binlog文件（默认binlog达到指定大小时才会生成新的binlog文件）

1、service mysqld restart
2、mysql> flush logs;
3、mysql  -e  "flush logs"
4、mysqldump -uroot -p123 --flush-logs db100 > test.sql
5、mysqladmin -uroot -p123 flush-logs
  


000010
000011  <-----
四、用binlog日志实现增量备份

binlog 文件记录sql语句的方式
时间点      time
字符偏移量  position
mysqlbinlog  [选项]    binlog文件名  | mysql  -uroot -p123

mysqlbinlog  --start-position=节点值    binlog文件名
选项
 --start-position=节点值
 --stop-position=节点值

 --start-datetime="yyyy-mm-dd  hh:mm:ss"
 --stop-datetime="yyyy-mm-dd  hh:mm:ss"



mysqlbinlog  --start-datetime="2014-01-06 14:50:51"  --stop-datetime="2014-01-06 14:51:47" mysqld-bin.000002  | mysql


第一个周一
crontab -e 
00 23  * * 1  mysqldum --flush-logs  db > /db-`data +%F`.sql

mysqld-bin.000001   db-2014-01-07.sql



第2个周一

mysqld-bin.000002   db-2014-01-14.sql


第3个周一
mysqld-bin.000003   db-2014-01-21.sql


mysqld-bin.000004








