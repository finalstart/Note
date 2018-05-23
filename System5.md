一、用户账户管理
1，用户账户：
	超级用户root（0）
	程序用户（1~499）
	普通用户（500~60000）
   组账户：
	基本组（私有组）
	附加组（公共组）
2，/etc/passwd	保存用户账户的基本信息
# grep student /etc/passwd
student:x:500:500::/home/student:/bin/bash
	字段1：帐号名student
	字段2：密码占位符x
	字段3：UID
	字段4：GID
	字段5：用户全名
	字段6：宿主目录
	字段7：登录Shell   /sbin/nologin不允许登录shell
3，/etc/shadow	保存密码字串/有效期等信息
# grep student /etc/shadow	
student:$1$po/zD0XK$4HSh/Aeae/eJ6dNj1k7Oz1:14495:0:99999:7:::
	字段1：帐号名称
	字段2：加密的密码
	字段3：从1970/1/1到上次修改密码的天数
	字段4：多少天之内不能修改密码，默认值为0
	字段5：密码的最长有效天数，默认值为99999
	字段6：密码过期之前警告天数，默认值为7
	字段7：密码过期之后账户宽限时间 3+5
	字段8：帐号失效时间，默认值为空
	字段9：保留字段（未使用）
密码过期：一旦超过密码过期日期，用户成功登陆，Linux会强迫用户设置一个新密码，设置完成后才开启Shell程序
账户过期：若超过账户过期日期，Linux会禁止用户登陆系统，即使输入正确密码，也无法登陆
4,useradd	添加账户
	-u：指定 UID 标记号
	-d：指定宿主目录，缺省为 /home/用户名
	-e：指定帐号失效时间
	-g：指定所属的基本组（组名或GID）
	-G：指定所属的附加组（组名或GID）
	-M：不为用户建立并初始化宿主目录
	-s：指定用户的登录Shell

5，主要文件
	/etc/skel		新建账户模板文件
	~/.bash_profile		每次登录时执行
	~/.bashrc		每次进入新的Bash环境时执行
	~./bash_logout		每次退出登录时执行
	全局
	/etc/bashrc
	/etc/profile
		
6，passwd	设置密码
	-d	清空密码
	-l	锁定用户账户
	-S	查看用户账户的状态
	-u	解锁用户账户
	--stdin	从标准输入取密码

7，usermod	修改用户账户的属性
	-l	更改用户账户登录名称
	-L	锁定
	-U	解锁
	-u、-d、-e、-g、-G、-s与useradd相同

8，userdel	删除账户
	-r	连同主目录一起删除
将系统中创建的普通账户全部删除

...
9，/etc/login.defs	密码有效期控制

10，/etc/default/useradd 
11，chage		
	-l	列出密码有效期
	-d					3
	-m	指定密码最小天数			4
	-M	指定密码最大天数			5
	-W	警告时间				6
	-I	指定密码失效后多少天锁定账户	7
	-E	指定账户过期时间 YYYY-MM-DD	8
10，id		查看用户ID标识
	-u	输出用户数字标识
	-g	输出用户所在基本组的数字标识
	-G	输出用户所在附加组的数字标识
	-n	现实对应项的名称	


二、组账户管理
1，/etc/group	组账号文件
# grep student /etc/group
	字段1：组账户名称
	字段2：密码占位符x
	字段3：组账户GID号
	字段4：本组的成员用户列表
2，/etc/gshadow	组账号管理信息
# grep student /etc/gshadow
	字段1：组账户名称
	字段2：加密后的密码字符串
	字段3：本组的管理员列表
	字段4：本组的成员用户列表
3，groupadd	添加组账户
	-g	指定gid

4，gpasswd	设置组密码和组管理
	-A	定义组管理员列表
	-a	添加组成员
	-d	删除组成员
	-M	定义组成员列表，可设置多个
	-r	移除密码

5，newgrp	切换所属基本组

6，groupdel	删除组

7，groups	查看用户组信息
三、源码包的编译安装
1，准备开发环境gcc、g++、make
2，源码安装基本过程
tar 		解包/usr/src
./configure	配置
make		编译
make install	安装



########################测试题############################
一、用户账户管理
1，查看/etc/passwd文件的第一行
[root@server1 ~]# head -1 /etc/passwd
root:x:0:0:root:/root:/bin/bash

2，查看/etc/shadow文件的第一行
[root@server1 ~]# head -n 1 /etc/shadow
root:$1$lMfdzZGT$zHQTo0sPK2nFVbH9R4jfq/:16058:0:99999:7:::

3，查看/etc/passwd、/etc/shadow、/etc/group、/etc/gshadow中与root相关的内容
[root@server1 ~]# cat /etc/passwd /etc/shadow /etc/group /etc/gshadow | grep root

4，创建账户
                student
                stu01，宿主目录设为/opt/stu01
                stu02，uid为10001，账户在2015-06-30号过期，基本组设为stu01
                sys01，不用于登录
	sys02，不创建宿主目录

[root@server1 ~]# useradd student
[root@server1 ~]# useradd stu01 -d /opt/stu01
[root@server1 ~]# useradd stu02 -u 10001 -g stu01 -e 20150630
[root@server1 ~]# useradd sys01 -s /sbin/nologin
[root@server1 ~]# useradd sys02 -M

5，查看用户模板文件里面的内容，包括隐藏文件
[root@server1 skel]# cd /etc/skel/
[root@server1 skel]# ls -a
.  ..  .bash_logout  .bash_profile  .bashrc  .emacs  .mozilla

6，对当前root账户设置alias，baibai等于关机命令，使之永久生效

7，针对student操作
                设置密码为123456，然后用student登录自己修改密码
                清空student的密码，查看/etc/shadow里面与student相关的内容
                锁定student的密码（用2种方法），查看/etc/shadow里面与student相关的内容
                查看student的状态
                解锁student的密码（用2种方法），查看/etc/shadow里面与student相关的内容
8，针对stu01操作
                从标准输入给stu01设置密码为redhat   --stdin
                修改stu01的登录名为kaka
9，修改stu02的uid为800，宿主目录为/opt/stu02，过期时间为2017-06-30，基本组为sys01
10，修改sys01可以登陆系统
11，把/etc/login.defs里面非以#号开头的和非以空格行显示出来，这个文件的作用
12，列出stu02的账户有效期
13，设置kaka账户过期时间为2015-10-01
14，设置sys01密码失效3天后锁定账户，密码最小天数为1天，最大天数为30天
15，查看当前用户的ID标识信息
16，查看stu02的ID标识信息


二、组账户管理
1，查看/etc/group文件的最后一行
2，查看/etc/gshadow文件的最后一行
3，创建stugrp这个组，GID为600
4，将student，kaka，sys01加入stugrp组
5，设置stu01为stugrp组管理员
6，将sys01从stugrp组中删除
7，设置stugrp组密码为123456，清除密码。再设置为组密码为redhat
8，创建账户mike，用mike登陆，加入到stugrp组中
9，删除stugrp组
10，查看student与kaka的组信息

三、源码包的安装
1，安装apache源码包



一、手动创建用户的过程 （禁止useradd）
1、/etc/passwd
   /etc/shadow
   /etc/group
   /etc/gshadow
2、/home/xxxx
3、/var/spool/mail/xxxx
4、/etc/skel/.*
5、权限

二、迁移账户宿主目录(/home空间不足)
old    /home
new    /data/home