

mysql集群
客户端         client（访问数据的）
管理集群的主机 mgm（管理集群中的所有主机）
SQL节点        sql(供用户访问和执行sql语句)
数据节点       ndbd(存储表中的记录)


mysql-cluster


                     client
 
                           mgm   192.168.1.1
              
                 10         20  
               sql_A       sql_B   

                
               ndbd_A      ndbd_B     
                 30         40

公共配置
1 ping 通
2 service iptables  stop  ;  setenforce 0
3 service mysqld  stop;ckconfig --level 35 mysqld off
4 useradd mysql
5 安装集群软件


配置管理主机:（管理集群中的所有主机） 
启动管理进程时调用配置文件/usr/local/cluster/config.ini 知道集群中主机的角色。

vim   /usr/local/cluster/config.ini （不允许用空行）
[ndbd default]
NoOfReplicas=2
DataMemory=80M
IndexMemory=18M
[ndb_mgmd]
id=1
hostname=192.168.1.1
datadir=/usr/local/cluster/ndbddata
[ndbd]
id=30
Hostname=192.168.1.30
Datadir=/usr/local/cluster/ndbddata
[ndbd]
id=40
Hostname=192.168.1.40
Datadir=/usr/local/cluster/ndbddata
[mysqld]
Id=10
Hostname=192.168.1.10
[mysqld]
Id=20
Hostname=192.168.1.20


配置数据节点(ndbd)存储表中的记录
mkdir  /usr/local/cluster/ndbddata

mv /etc/my.cnf /etc/my.cnf.bak

vim /etc/my.cnf
[mysqld]
datadir=/usr/local/cluster/ndbddata
ndb-connectstring=192.168.1.1

[mysql_cluster]
ndb-connectstring=192.168.1.1


配置sql节点:供用户访问 、执行sql语句
mysql -hlocalhost -uroot
mysql> show databases;

1、编写配置文件
mv /etc/my.cnf /etc/my.cnf.bak
vim  /etc/my.cnf
[mysqld]
ndbcluster
ndb-connectstring=192.168.1.1
[mysql_cluster]
ndb-connectstring=192.168.1.1

2、初始化授权库
cd /usr/local/cluster/
./scripts/mysql_install_db --user=mysql

ls data/mysql


启动管理主机的管理进程
cd /usr/local/cluster/bin
./ndb_mgmd -f ../config.ini

[root@mgm bin]# pgrep mgmd
3147
[root@mgm bin]# pkill -9 mgmd

启动数据节点的数据进程
cd /usr/local/cluster/bin
./ndbd
[root@mail01 bin]# pgrep ndbd
8731
8732
[root@mail01 bin]# pkill -9 ndbd


启动sql节点的数据库服务
cd /usr/local/cluster/bin
mkdir -p /usr/local/mysql/data
touch  /usr/local/mysql/data/sql-A.err
[root@sql-A bin]# ./mysqld_safe --user=mysql &


集群里的表要使用ndbcluster存储引擎
sql:
mysql -uroot
create  table 数据库名.表名(字段列表)engine=ndbclust;











