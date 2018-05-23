一、引导过程及服务
1，启动引导
        主机加电自检，加载BIOS硬件信息
        读取MBR的引导文件（grub，lilo）
        引导linux内核
        运行第一个进程init（进程号永远为1）
        进入相应的运行级别
        运行终端，输入用户名密码
2，init进程与配置文件    
        加载执行/sbin/init程序，进程ID始终为1。
        主配置文件/etc/inittab
        18行：id:5:initdefault:		设置系统默认启动的级别
        32行：ctrlaltdel			三键重启
文件语法
[设置项目]:[runlevel]:[init操作行为]:[命令项目]
设置项目：最多4个字符，表示init工作项目，简单说明
运行级别：0123456
init操作：
	initdefault    表示默认启动级别
	sysinit	    初始化操作
	ctrlaltdel  	    三键重启
	wait	    表示后面接的项目必须执行完毕才能继续随后操作
	respawn	    表示后面接的init认为主动重新启动
     修改配置文件后，下次开机生效；如果想立即生效，使用命令 init q
命令项目：通常是一些脚本
3，Linux运行默认级别
             0    关机
             1    单用户（single）
             2    多用户（但是不支持网络账户登录）
             3    服务器模式
             4    保留，未使用
             5    图形
             6    重启
默认级别千万不要设置为 0 4 6
4，切换运行级别
	init [0-6]
5，runlevel        查看当前运行级别
N    5
N        上一次运行级别（N表示上一次没级别,S--------->1）
5         当前运行级别  
6，其他初始化配置
        /etc/rc.d/rc.sysinit 系统初始化
        /etc/rc.d/rc	     指定运行级别，加载终止相应系统服务
        /etc/rc.d/rc.local   开机自动运行脚本
 
二、服务管理
1，服务状态
        chkconfig --list		显示全部服务的启动状态
        chkconfig --list xxx	显示xxx服务的启动状态
2，开启独立服务（/etc/init.d/）
自动
      chkconfig --level 0~6 服务名 动作(on/off)（下次启动生效）
      ntsysv --level 级别
手动
      service 服务名 start/stop/restart/status(当前级别生效)
      /etc/init.d/服务名 start/stop/restart/status/reload(当前级别生效)
3，开启非独立服务（/etc/xinetd.d/）
      chkconfig 服务名 on/off
      service xinetd restart

三、进程管理
1， 程序：硬盘中的代码
      进程：代码执行产生
2，ps查看进程
ps    显示某一时刻进程状态信息，静态
    	-A和-e一样	显示所有的进程	
常用命令选项
	a：显示当前终端下的所有进程
	u：使用以用户为主的格式输出信息
	x：显示当前用户在所有终端下的进程
	-e：显示系统内所有的进程
	-l：使用长格式输出信息
	-f：以最完整的的格式输出信息
ps aux    列出正在运行的所有进程
常见的STAT状态指示
        R：正在运行
	S：处于休眠状态，在需要时可唤醒
	D：不可中断的休眠，通常为等待I/O的情况
	T：停止状态，被挂起或跟踪的情况
	Z：僵尸状态，进程已终止但内存无法释放
ps -elf     以长格式显示系统中所有的进程信息
            	F：权限标记，4对应root、1表示仅复制无执行权限
            	PPID：父进程的PID号
ps -l   ppid 父进程号
3，top动态查看进程
	？：查看帮助（列出可用的按键指令）
	P、M：根据 %CPU、%MEM 降序排列
	T：根据进程消耗的 TIME 降序排列
	k、r：杀死指定的进程、重设进程优先级
	q：退出 top 程序

4，pgrep	 根据特定条件
	-l	PID连同进程名一起输出
	-U	检索指定用户的进程
	-t	检索指定终端的进程		
5，pstree 树状结构
	-a	显示完整命令行
	-u	列出各进程所属的用户名
	-p	列出对应的PID号
6，进程控制
	&	放入后台运行，运行在内存中的进程
        ctrl +z	将当前的作业放入后台并暂停运行
        jobs		查看后台进程
        fg 编号	把后台进程调到前台
        bg 编号	让程序在后台运行
7，终止进程
        ctrl +c	终止当前正在运行的进程
        kill -9 pid	强制杀掉进程
        killall 	进程名
        pkill   	进程名



########################测试题######################
一、引导过程及服务
1，将init主配置文件里面以#号开头并且是空行的内容过滤掉，显示剩下内容
[root@localhost ~]# grep -vE "^#|^$" /etc/inittab

2，查看当前运行级别
[root@localhost ~]# runlevel

3，切换到服务器级别
[root@localhost ~]# init 3

4，设置以后Linux每次启动自动进入服务器级别并且屏蔽ctrl + alt + del重启
[root@localhost ~]# vim /etc/inittab
id:3:initdefault:
#ca::ctrlaltdel:/sbin/shutdown -t3 -r now

二、服务管理
1、安装vsftpd服务，启动该服务。确保无误自启
[root@localhost ~]# rpm -q vsftpd
[root@localhost ~]# find /misc/cd/Server -name vsftpd*
/misc/cd/Server/vsftpd-2.0.5-28.el5.x86_64.rpm
[root@localhost ~]#cd /misc/cd/Server
[root@localhost Server]# rpm -ivh vsftpd-2.0.5-28.el5.x86_64.rpm 
[root@localhost Server]# rpm -ql vsftpd | grep init.d
/etc/rc.d/init.d/vsftpd
[root@localhost Server]# service vsftpd start
[root@localhost Server]# service vsftpd status
[root@localhost Server]# lftp 127.0.0.1
lftp 127.0.0.1:~> ls                
drwxr-xr-x    2 0        0            4096 Sep 25  2012 pub

[root@localhost ~]# chkconfig --list vsftpd
vsftpd          0:关闭  1:关闭  2:关闭  3:关闭  4:关闭  5:关闭  6:关闭
[root@localhost ~]# chkconfig --level 35 vsftpd on
[root@localhost ~]# chkconfig --list vsftpd
vsftpd          0:关闭  1:关闭  2:关闭  3:启用  4:关闭  5:启用  6:关闭

2、安装telnet服务，启动该服务。
[root@localhost Server]# rpm -q telnet-server
package telnet-server is not installed
[root@localhost Server]# find -name telnet-server*
./telnet-server-0.17-41.el5.x86_64.rpm
[root@localhost Server]# rpm -ivh telnet-server-0.17-41.el5.x86_64.rpm 
[root@localhost Server]# rpm -ql telnet-server | grep init.d
[root@localhost Server]# rpm -ql telnet-server | grep xinetd.d
/etc/xinetd.d/telnet
[root@localhost Server]# chkconfig telnet on
[root@localhost Server]# service xinetd status
xinetd (pid  3820) 正在运行...
[root@localhost Server]# telnet 127.0.0.1
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
Red Hat Enterprise Linux Server release 5.9 (Tikanga)
Kernel 2.6.18-348.el5 on an x86_64
login: 


3、查看防火墙(iptables)服务的开机启动状态，将该服务自启修改成关闭
[root@localhost ~]# chkconfig --list iptables
iptables        0:关闭  1:关闭  2:启用  3:启用  4:启用  5:启用  6:关闭
[root@localhost ~]# chkconfig iptables off
[root@localhost ~]# chkconfig --list iptables
iptables        0:关闭  1:关闭  2:关闭  3:关闭  4:关闭  5:关闭  6:关闭

三、进程管理
1，使用top命令查看当前进程，分别以CPU MEM使用率降序排列
top
P
M

2，显示log相关的进程并显示其进程名
[root@localhost ~]# pgrep -l "log"
3442 syslogd
3445 klogd

3，树状结构查看当前进程情况并显示进程id
[root@localhost ~]# pstree -p


4，安装xsnow包（找项目经理获取）
[root@localhost Desktop]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# ls
rhel-debuginfo.repo
[root@localhost yum.repos.d]# cp rhel-debuginfo.repo server.repo
[root@localhost yum.repos.d]# vim server.repo 
[root@localhost yum.repos.d]# yum list |wc -l
3347
[root@localhost yum.repos.d]# cd -
/root/Desktop
[root@localhost Desktop]# ls
xsnow-1.42-10.i386.rpm
[root@localhost Desktop]# yum -y localinstall xsnow-1.42-10.i386.rpm --nogpgcheck

5，执行xsnow命令，放入后台运行，查看后台任务
xsnow  &
jobs
6，将后台运行的xsnow调到前台，在放入后台暂停运行
[root@localhost Desktop]# fg 1
Ctrl+z
7，使xsnow在后台继续运行
[root@localhost Desktop]#bg1

8，查看xsnow pid，kill掉该进程
[root@localhost Desktop]# pgrep -l "xsnow"
5230 xsnow
[root@localhost Desktop]# kill 5230
