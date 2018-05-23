一、基本权限和归属
1，访问权限
读取：允许查看、显示目录列表
写入：允许修改，允许在目录中新建、移动、删除文件或子目录
可执行：允许运行程序、切换目录
2，查看文件的权限
# ls -l install.log
-|rw-|r--|r-- 1  root  root  26195 Dec 17 10:42  install.log
① ② ③  ④ ⑤   ⑥    ⑦     ⑧       ⑨            ⑩

①,文件类型
-代表普通文件
d代表目录
l代表连接
②,rw-:代表文件所有者的权限(u)
r=读=4
w=写=2
x=执行=1
③,r--：代表文件所属组的权限(g)
r=读=4
w=写=2
x=执行=1
④,r--：其他用户的权限(o)
r=读=4
w=写=2
x=执行=1
a=ugo
⑤，文件  硬链接数
    目录  该目录下有多少子目录包括.和..
⑥，文件所有者
⑦，文件所属组
⑧，文件大小
⑨，文件修改时间
⑩，文件名

3，命令 （R表示递归）
chmod 改变权限
chmod ugoa [+-=] [rwx] 文件
chmod 数字 文件

文件最大权限666
目录最大权限777
默认创建文件的权限644
默认创建目录的权限755
umask 
最大权限的rwx-umask的rwx=默认权限
补充：
对于目录来说没有x权限，无论有没rw，都不能进入该目录
umask -S

chown 改变所有者与所属组
chown  所有者：所属组  对象
chgrp   组 文件	更改文件的属组


二、附加权限控制
1，特殊权限介绍
Set UID：	4	User+x
Set GID：	2	Group+x
Sticky Bit：	1	Other+x
2，特殊权限作用
Set UID：
	只能对可执行程序设置，当其他用户执行带SUID标记的程序时 ，将会使用程序所有者的身份去执行

Set GID：
	可以对可执行程序设置，当其他用户执行带SGID标记的程序时 ，将会使用程序所属组的身份去执行

	可以对目录设置，当对目录设置SGID后，任何人在该目录下创 建文件和目录的所属组自动继承该目录的所属组

Sticky Bit：
	对目录设置，任何人在该目录下创建文件和目录，只有root与 文件创建者有删除权限

3，ACL策略
创建账户：mike  john  kaka
创建文件：/data/file1.txt
1）mike对文件有读写权限，john只有读权限。其他用户没有任何权限
2）kaka具有与john相同权限
3）创建lily，lily对file1.txt具有读执行权限，其他用户没有任何权 限

getfacl		查看ACL策略
setfacl		设置ACL策略
	-m	定义一条ACL策略
	-x	删除指定的ACL策略
	-b	清除所有已设置的ACL策略
	-R	递归设置
	-d	为目录设置默认权限


########################测试题############################
一、基本权限和归属
1，长格式显示/etc/passwd和/boot，分别说出他们相应的权限
[root@localhost tmp]# ls -ld /boot /etc/passwd
drwxr-xr-x 4 root root 1024 07-10 17:22 /boot
-rw-r--r-- 1 root root 1966 12-09 16:36 /etc/passwd

2，用字符权限形式将Desktop（包括里面的文件）权限更改成770
[root@localhost Desktop]# chmod u+rwx,g+rwx,o-rwx /root/Desktop/ -R

3，用数字权限形式将Desktop（包括里面的文件）权限更改成755
[root@localhost Desktop]# chmod 755 /root/Desktop/ -R

4，编写test.sh脚本，里面内容是 echo Hello World。设置允许执行该脚本
[root@localhost ~]# vim hello.sh

#!/bin/bash
echo hello world!

[root@localhost ~]# ll hello.sh 
-rw-r--r-- 1 root root 30 12-09 19:53 hello.sh
[root@localhost ~]# chmod u+x hello.sh 
[root@localhost ~]# ./hello.sh 
hello world!

5，查看umask值，代表什么意思
[root@localhost Desktop]# umask
0022
#对于文件的完整权限则是666,目录的最大权限是777，现在umask的值为0022，
#对文件 	(rw-rw-rw-)-(----w--w-)=rw-r--r--  即644
#对目录    (rwxrwxrwx)-(----w--w-)=rwxr-xr-x  即755
修改umask值
[root@localhost Desktop]# umask 0033


6，创建一个/opt/studir目录，权限为755。将/opt/studir目录所有者改成student，所属组为users
[root@localhost Desktop]# mkdir /opt/studir
[root@localhost Desktop]# chmod 755 /opt/studir/        或创建时直接设置权限 mkdir -m 755 /opt/studir
[root@localhost Desktop]# useradd student
[root@localhost Desktop]# groupadd users
groupadd：users 组已存在
[root@localhost Desktop]# chown student:users /opt/studir/

7，在将/opt/studir目录包括目录下的所有文件和子目录所有者改成stu01
[root@localhost Desktop]# chown stu01 /opt/studir/ -R

8，删除实验中创建的账户与组
[root@localhost Desktop]# userdel -r student
[root@localhost Desktop]# userdel -r stu01

二、附加权限控制
1，长格式显示/tmp和/usr/bin/passwd，查看其权限
[root@localhost Desktop]# ls -ld /tmp /usr/bin/passwd 
drwxrwxrwt 7 root root  4096 12-09 17:07 /tmp
-rwsr-xr-x 1 root root 27936 2010-08-03 /usr/bin/passwd

2，创建普通账户tom，查找mkdir命令在什么地方，将其拷贝到/bin下改名为smkdir，给其设置suid权限
[root@localhost Desktop]# useradd tom
[root@localhost Desktop]# which mkdir
/bin/mkdir
[root@localhost Desktop]# cp /bin/mkdir /bin/smkdir
[root@localhost Desktop]# chmod u+s /bin/smkdir 
[root@localhost Desktop]# ll /bin/smkdir 
-rwsr-xr-x 1 root root 31664 12-09 17:33 /bin/smkdir

3，tom登陆使用mkdir创建/tmp/test1目录，查看其权限
[root@localhost ~]# su - tom
[tom@localhost ~]$ mkdir /tmp/test1
[tom@localhost ~]$ ls -ld /tmp/test1
drwxrwxr-x 2 tom tom 4096 12-09 17:59 /tmp/test1

4，tom登陆使用smkdir创建/tmp/test2目录，查看其权限
[tom@localhost ~]$ smkdir /tmp/test2
[tom@localhost ~]$ ls -ld /tmp/test2
drwxrwxr-x 2 root tom 4096 12-09 18:01 /tmp/test2

5，删除smkdir命令
[root@localhost ~]# which smkdir
/bin/smkdir
[root@localhost ~]# rm /bin/smkdir 
rm：是否删除 一般文件 “/bin/smkdir”? y

6，查找cp命令在什么地方，将其拷贝到/bin下改名为gcp，给其设置sgid权限
[root@localhost ~]# which cp
alias cp='cp -i'
        /bin/cp
[root@localhost ~]# cp /bin/cp /bin/gcp
[root@localhost ~]# chmod g+s /bin/gcp

7，用tom账户登录，使用cp命令创建/tmp/file1文件。使用gcp命令创建/tmp/file2文件，查看这2个文件权限部分区别。
[tom@localhost ~]$ cp test /tmp/file1
[tom@localhost ~]$ gcp test /tmp/file2
[tom@localhost ~]$ ll /tmp/
-rw-rw-r-- 1 tom  tom     0 12-09 18:07 file1
-rw-rw-r-- 1 tom  root    0 12-09 18:11 file2

8，删除gcp命令，tom账户
[root@localhost ~]# rm /bin/gcp
rm：是否删除 一般文件 “/bin/gcp”? y
[root@localhost ~]# userdel -r tom

9，创建user01，user02账户，group01组。/software目录，用user01在该目录下创建test01，
user02在该目录下创建test02，查看其权限
[root@localhost ~]# useradd user01
[root@localhost ~]# useradd user02
[root@localhost ~]# groupadd group01
[root@localhost ~]# mkdir /software
[root@localhost ~]# ls -ld /software/
drwxr--r-- 2 root root 4096 12-09 18:21 /software/
[root@localhost Desktop]# su - user01
[user02@localhost ~]$ mkdir /software/test01
mkdir: 无法创建目录 “/software/test01”: 权限不够
[root@localhost Desktop]# su - user02
[user02@localhost ~]$ mkdir /software/test02
mkdir: 无法创建目录 “/software/test02”: 权限不够


10，更改software目录所有者为root，所属组为group01，允许所有人对此目录有读、写、执行权限，
并且在该目录下创建文件和目录都自动继承software的属组
[root@localhost ~]# chown root:group01 /software/
[root@localhost ~]# ll -d /software/
drwxr--rwx 4 root group01 4096 12-09 18:26 /software/
[root@localhost ~]# chmod 777 /software/ -R
[root@localhost ~]# ll -ld /software/*
drwxrwxrwx 2 user01 user01 4096 12-09 18:25 /software/test01
drwxrwxrwx 2 user02 user02 4096 12-09 18:26 /software/test02
[root@localhost ~]# chmod g+s /software/


11，删除user01，user02账户，group01组，以及/software目录
[root@localhost ~]# userdel -r user01
[root@localhost ~]# userdel -r user02
[root@localhost ~]# groupdel group01
[root@localhost ~]# rm -rf /software/

12，查看当前磁盘分区情况，查看根分区是否支持acl
[root@localhost ~]# tune2fs -l /dev/sda1 | grep acl
Default mount options:    user_xattr acl

13，查看/root的acl权限，创建账户jack，jack是否可以进入/root？为什么
[root@localhost ~]# getfacl /root
getfacl: Removing leading '/' from absolute path names
# file: root
# owner: root
# group: root
user::rwx
group::r-x
other::---

14，使用acl权限设置jack能够进入/root
[root@localhost ~]# setfacl -m u:jack:x /root
[root@localhost ~]# getfacl /root
getfacl: Removing leading '/' from absolute path names
# file: root
# owner: root
# group: root
user::rwx
user:jack:x
group::r-x
mask::rwx
other::---

15，创建/data目录，设置在/data目录下新建的文件和目录，jack都有读、写、执行权限
[root@localhost ~]# mkdir /data
[root@localhost data]# setfacl -dm u:jack:rwx /data 

16，清除/data的acl权限
[root@localhost ~]# setfacl -b /data

17，删除/data目录与jack账户
[root@localhost ~]# userdel -r jack
[root@localhost ~]# rm -rf /data
