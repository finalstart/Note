一、修改表结构（修改表结构会影响表里已有的记录）
alter  table  数据库名.表名 处理动作；

处理动作：
添加新字段  add
默认新添加的字段追加在已有字段末尾
first   把新添加的字段添加所有字段的上方
after  字段名   把新字段添加在指定字段名的下方

add  字段名  字段类型（宽度） 约束条件 first;


add  字段名  字段类型（宽度） 约束条件 after 字段名,
add  字段名  字段类型（宽度） 约束条件,
add  字段名  字段类型（宽度） 约束条件;

删除字段： drop
drop  字段名；

drop 字段名，drop  字段名；


修改字段类型 modify
modify  字段名   新字段类型（宽度） 约束条件；

modify  字段名1   新字段类型（宽度） 约束条件，
modify  字段名2   新字段类型（宽度） 约束条件，
modify  字段名3   新字段类型（宽度） 约束条件；

更改字段名  change 

change   源字段名  新字段名  字段类型(宽度) 约束条件；


change   源字段名  新字段名  字段类型(宽度) 约束条件，
change   源字段名  新字段名  字段类型(宽度) 约束条件，
change   源字段名  新字段名  字段类型(宽度) 约束条件；


修改表名
alter  table  源表名  rename  to  新表名；

复制表
create  table  新表名  SQL查询语句；

复制源表的表结构
create  table  新表名  没有SQL查询的语句；

二、mysql索引类型
1.什么是索引（相当与书的目录）

100页 （内容）

1-5 目录   6-100

15~20  Apache 新内容 

2.索引优缺点？
加快查询表记录的速度
会减慢对表记录写（insert update  delete）的速度
所用会占用物理磁盘空间
stu_info_1.frm      表结构
stu_info_1.MYD      表里数据
stu_info_1.MYI  保存表索引信息文件

3.怎么查看表里是否有索引字段
desc  表名;(KEY)

4.怎么看表的索引信息
show  index  from  表名；

show index  from  mysql.user;
索引类型 索引字段  索引算法(BTREE)
                            B+TREE  hash
BTREE(二叉数算法)
            1-60
      1-30          31-60
   1-15         16-30
1-7.5    7.6-15  
1-3.725  

5.mysql索引类型
普通索引  index   *
唯一索引  unique
主    键  primary key   *
外    键  foreign  key   *
全文索引  fulltext


普通索引的使用？
把经常做查询条件的字段设置为INDEX字段
select name,age,sex,tel from stuinfo where sex="boy";

1 建表时就创建索引字段
create  table  t20(
name varchar(10),
age int(2),
sex  enum("boy","gril") default "boy",
index(name)
);

create  table  t21(
name varchar(10),
age int(2),
sex  enum("boy","gril") default "boy",
index(name),
index(sex)
);

create  table  t22(
name varchar(10),
age int(2),
sex  enum("boy","gril") default "boy",
index(name,sex)
);



2 在已有的表里添加index字段
create  index 索引名 on  表名(字段名列表)；

删除索引
drop index 索引名  on 表名;


使用 唯一索引  unique
create table  t31(
stu_id  char(4),
name  char(4),
age int(2),
unique(stu_id),index(name)
);

create  unique  index  索引名  on 表名(字段名列表);

drop index 索引名  on 表名;


主    键  primary key 
如何使用主键？

1
create table  t40(
id  int(2) primary key  auto_increment,
name varchar(10),
age int(2)
);

create table  t50(
id  int(2)  auto_increment,
name varchar(10),
age int(2),
primary key(id)
);


mysql> alter table t30 add primary key(stu_id);

mysql> alter table t40 drop primary key;


primary key
ip           port     sername     status

create table  sername(
id  varchar(15),
port varchar(5),
sername varchar(10),
status  enum("deny","allow") default "deny",
primary key(id,port)
);

普通索引 index 
唯一索引 unique
主    键 primary key

创建 查看  删除  使用

drop  table  表名；
练习
1
在yg_tab表里添加名id的字段，在所有字段的上方，为主键盘字段且有
自动增长功能。把表中的name和sex字段设置为index字段.
在部门字段下方添加mail字段默认值是user@tarena.com
在id字段下添加名为yg_id字段，字段值不允许重复，
在mail字段下方添加loves字段保存员工的爱好 默认爱好 IT,film
表中所有字段不允许插入NULL值 
2
查看表结构看设置是否正确

3
向表中插入3条新的员工记录

4 查看记录是否插入成功

5 删除表里所有字段的索引属性

6 查看表结构看是否删除成功。

7 复制当前表 新表名为 newuser

8 删除yg_tab表里的所有记录。



使用外键  foreign  key

http://www.51cto.com/bbs

数据库里 dbname.reg_tab(loginname,password)
         dbname.page_tab(发帖人 帖子标题 帖子内容 发帖时间)
                    

登陆名 tarean1
密码   888888

foreign    key(当前表里字段名)  References  表名(表名里的字段名)  
on   update   cascade    on  delete  cascade


1 表的存储引擎要是innodb存储引擎。
2 外键字段的类型要匹配
3 被参照字段要有明确的索引（index primary key）



三、mysql存储引擎
硬件 -> 内核  -> shell  -> 用户


1、mysql体系结构
由8部分组成：连接池、Sql接口、分析器、优化器、缓存和缓冲、存储引擎、管理工具、物理存储设备

1、连接池：进程数限制、内存检查、缓存检查等

2、SQL接口：用户通过sql客户端发过来的命令，由sql接口接收，sql操作有：
           DML数据操作语言：查询，修改，升级数据等
           DDL数据定义语言：创建一个新的数据库，新的索引，删除一个用户等
                                      存储过程  视图触发器
3、分析器： 分析查询语句 事务处理 对象访问权限

4、优化器： 优化访问路径 、 生成执行树

5、缓存和缓冲 保存sql查询结果

6、管理工具：备份，恢复，安全，移植，集群等，这些工具一般和文件系统打交道，不需要和mysql-server打交道，它们对应的都是命令。
7、存储引擎：用于管理文件系统，将逻辑结构转换为物理结构的程序。负责为数据库执行实际的数据I/O操作。
不同的存储引擎有不同的功能和存储方式?

8、物理存储设备(文件系统)


mysql> show engines;

mysql> show variables like "table_type";

create  table  表名(字段列表)engine=存储引擎名；

create table studb.t200(id int)engine=innodb;


alter  table  表名  engine=存储引擎名；


default-storage-engine=innodb


yg_tab
yg_id    员工编号
yg_name  员工姓名

create  table  studb.yg_tab(
yg_id int(2) primary key  auto_increment,
yg_name  varchar(10) not null
)engine=innodb;



gz_tab
id     工资记录编号
gz_id  员工编号
gz     员工工资

create  table studb.gz_tab(
id int(2) primary key auto_increment,
gz_id  int(2) not null,
gz  float(7,2),
foreign key(gz_id)  references  yg_tab(yg_id) on update cascade 
on delete cascade )engine=innodb;




insert into studb.yg_tab(yg_name)values("jim");

insert into studb.yg_tab(yg_name)values("lucy");

insert into gz_tab(gz_id,gz)values(1,10000.99);

insert into gz_tab(gz_id,gz)values(4,5000.88);


mysql> update yg_tab set yg_id=8 where yg_id=2;


父表
子表
一父多子

17:30
把课堂的例子实现
一子多父（不同的字段参考不同的父表  相同字段参考不同的父表）
外键字段可不可以被删除
删除字段的foreign key  属性

myisam
默认使用的存储引擎
独享表空间
不支持事务和事务回滚
支持表锁


innodb
共享表空间
支持事务和事务回滚
支持行锁

事务？

cardA              cardB
       
5000  ------------->
select * from tab1;
delete from tab1 
insert into cardB.tabB
select from cardB.tabB



事务回滚?


锁机制？（解决用户的并发访问问题）

锁类型
写锁（delete  update  insert） 互斥锁 排它锁
读锁 (select  desc  show )     共享锁

脏读
幻读

锁粒度
表锁(myisam)  并发访问量低   系统开销小 
行锁(innodb)  并发访问量高   系统开销大
 













