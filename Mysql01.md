1 了解提供数据库服务的软件有哪些？
Oracle  access   vf   	sql-server   
mysql   DB2   Postgresql    Sybase


Oracle   DB2    sql-server   mysql

商业软件   Oracle   DB2    sql-server
开源软件   mysql    Postgresql

跨平台(操作系统) LINUX   windows
Oracle   DB2   mysql    Postgresql

即跨平台有开源 ：mysql

2 数据库服务应用在哪里？
网站  论坛  银行  证券公司  购票系统  


mysql+web  (LAMP  LNMP)


NoSQL    DBMS
3 mysql软件有什么优点？
4 搭建数据库服务器
rpm包   rpm -ivh  xxx.rpm    yum -y install 软件名
源码包  ./configure  选项  make  make install

在ip是192.168.1.1服务器上搭建数据库服务。
1给服务器配置固定ip地址
2搭建本地yum源
3用yum安装提供数据库服务的软件包
yum -y install  mysql-server  mysql
4 启动数据库服务
service  mysqld  status|start|stop|restart

netstat  -utnalp | grep  :3306

进程名  mysqld
进程的所有者、所属组       mysql
端口号   3306
传输协议   tcp
主配置文件  /etc/my.cnf
数据库目录  /var/lib/mysql/

登陆数据库服务器
[root@mail01 ~]# mysql -hlocalhost -uroot
[root@mail01 ~]# mysql

姓名  性别  年龄  学历       班级   电话    QQ    mail   成绩
lxy   男     18    幼儿园     大1    110    
zy    女     18    本科      1310     999    xxx   xxxx

连接数据库服务器->选择一个库->选择一个表->把数据插入到表里

默认的3个库(首次启动数据库服务时创建的)
information_schema   虚拟库 
mysql                授权库
test                 公共库

/var/lib/mysql/mysql/

数据库服务器上的库 是以文件夹方式存放在数据库目录下。文件夹名与数据库名同名，表是以文件的形式存放在自己所在库对应的文件夹里,文件名与表名相同（默认情况下mysql会把一个表的数据用3个文件来保存）

5 数据库管理

show  databases;
create   database  数据库名；
drop  database  数据库名；
use  数据库名；
数据库名命名规则？

6 表管理(表存放在库里)

create  table  数据库名.表名(
<字段名  字段类型>[(宽度) 约束条件],
字段名  字段类型(宽度) 约束条件,
字段名  字段类型(宽度) 约束条件,
.......

);
desc  表名;
desc  数据库名.表名;
insert  into  数据库名.表名(字段名1，字段2，字段名n)
values
(字段名1的值，字段2的值，字段名n的值),
(字段名1的值，字段2的值，字段名n的值);


在studb数据库里创建名为stu_info表 存学生的年龄，

mysql数据类型（4）
1 数值类型  （年龄  体重  身高 工资  成绩）
  
  整数型（  小整数  大整数  极大整数）
            tinyint  
            1个字节  
            00000000  0
            11111111  255

mysql> create table t1(level tinyint unsigned);


  浮点型（带小数点的数字）  
  单精度 float(n,m)    n 表示数字位数个数 m 小数位个数  
         float(5,2) 000.00
         float(8,2) 000000.00
  双精度 double(n,m) n 表示数字位数个数 m 小数位个数

  10000.88
       0.00
  -1000.00

create  table  studb.t2(
gz  float(7,2)
);
99999.99


2 字符串类型（名字  家庭住址）
char  定长   0~255   字节
varchar  变长 0~65535字节


name1  char(3)
name2  varchar(3)


create  table  studb.t3(
name  char(3),
age   int(3)
);

create  table t6(
id int(4) zerofill,
id2 int zerofill
);




3 日期时间类型（生日 出生年份  注册时间）
年    year  
日期  date  
小时  time  
日期时间  datetime / timestamp


create  table  t7(
name varchar(10),
starday  year,
workday  date,
timework  time,
meetting  datetime
);

mysql> insert into t7 values ("lui",1987,"1990-10-08","09:30:00","2013-01-10 09:30:00");

mysql> insert into t7 
values 
("tom",1987,19901008,093000,20130110093000);


create  table t8(
time1  datetime,
time2  timestamp
);

1~69  20xx
70~99 19xx


4 枚举类型(字段的值只能在列举的范围内选择)
      （性别  爱好）
单选  enum("值1","值2","值N") 
多选  set("值1","值2","值N") 

create table t99(
name varchar(10) not null,
age  tinyint(2) unsigned  default 18,
sex  enum("boy","gril") not null default "boy",
likes  set("book","music","film","football") not null  default "book,film"
);

mysql> insert into t9  values ("jim",19,"boy","book,film");

NULL/null  表示字段没有值

课后练习：
在数据库ygdb里创建员工信息表yg_tab(保存员工的信息记录)
（员工姓名  性别   年龄   入职时间  职位   工作  部门） 
查看表结构
向表中插入3条记录
查看表中的记录


7 数据库导入导出
8 表记录的增删改查
9 用户授权 权限撤销
10 数据备份与恢复
11 mysql日志  
12 binlog日志的使用、
13 mysql主从同步
14 mysql读写分离
15 mysql集群
16 mysql调优






