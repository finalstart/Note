网络工程师
运维工程师、监控工程师、IDC工程师、Linux系统管理员、Linux系统工程师


一、Linux系统简介
1，GNU：编写大量兼容于UNIX系统的可自由传播使用的软件来替代UNIX系统中的商业软件
   GPL：通用公共许可证
	源代码免费开放
	可以执行，复制，再开发，学习，修改与强化自由软件
	不对使用自由软件的任何用户提供任何形式的责任担保或承诺
	可以出售（附带技术支持和服务）
   LGPL：次级公共许可证
	如果你对遵循LGPL的软件进行调用，而不是包含，则允许封闭源码

2，Linux发展
	1973年UNIX正式诞生
	1977年UNIX分支BSD诞生
	1991年Linux 0.02版发布
	1994年Linux1.0版发布
	1996年Linux2.0版发布
	2007年RHEL5发布
	2010年RHEL6发布
	吉祥物：企鹅


1. GPL就是Linux内核所采用的软件许可证，GPL的特点是：你拿人家的代码修改用了， 必须把修改后的代码公布。  所有的Linux都是采用的GPL许可，GPL许可允许GPL软件卖钱，但必须公布源码， 所以每个Linux发行版的代码都是全公开的，只是，使用这些代码的人必须也公开修改过的代码。  
2. Redhat的代码是公开的，但是他的二进制RPM包更新却不免费，这并不违反GPL许可。
3. 由于Redhat的源代码是公开的，所以CentOS项目的人拿来自己再编译，同样的代码， 同样的编译器，
编译出来的自然是同样的东西。只不过里面删除了Redhat的Logo以及相应信息，而核心的管理工具还是rpm，
只是用一个免费的软件包管理器yum（yellow dog update manager）替代了Redhat中的up2date，
up2date更新是连接到Redhat的收费服务站点的，通过钱买来的服务代码通过认证。 
4. 从品质上来说，CentOS从理论上应该跟Redhat一样的，毕竟是同样的源码。


     RHEL在发行的时候，有两种方式。一种是二进制的发行方式，另外一种是源代码的发行方式。
    无论是哪一种发行方式，你都可以免费获得(例如从网上下载)，并再次发布。但如果你使用了他们的
在线升级(包括补丁)或咨询服务，就必须要付费。
     RHEL一直都提供源代码的发行方式，CentOS 就是将 RHEL 发行的源代码从新编译一次，
形成一个可使用的二进制版本。由于 LINUX 的源代码是 GNU，所以从获得 RHEL 的源代码到编译成新的二进制，
都是合法。只是 REDHAT 是商标，所以必须在新的发行版里将 REDHAT 的商标去掉。
     REDHAT 对这种发行版的态度是：“我们其实并不反对这种发行版，真正向我们付费的用户，
他们重视的并不是系统本身，而是我们所提供的商业服务。”所以，CentOS 可以得到 RHEL 的所有功能，
甚至是更好的软件。但 CentOS 并不向用户提供商业支持，当然也不负上任何商业责任。
我正逐步将我的 RHEL 转到 CentOS 上，因为我不希望为 RHEL 升级而付费。当然，这是因为我已经有多年的 UNIX 使用经验，
因此 RHEL 的商业技术支持对我来说并不重要。但如果你是单纯的业务型企业，那么我还是建议你选购 RHEL 软件并购买相应服务。
这样可以节省你的 IT 管理费用，并可得到专业服务。
    一句话，选用 CentOS 还是 RHEL，取决于你所在公司是否拥有相应的技术力量。

3，Linux内核版本
# uname -r
2.6.18-348.el5
主版本号 2
次版本号 6
修订版号 18
红帽修订号 348.el5
次版本号：
	奇数：表示开发板
	偶数：表示稳定版

二、安装Linux操作系统
1，Windows磁盘分区与Linux磁盘分区举例
500G(SATA)
C	50G	/dev/sda1
D	50G	/dev/sda2
E	100G	/dev/sda5
F	100G	/dev/sda6
H	100G	/dev/sda7
I	100G	/dev/sda8
2，Linux下硬盘分区表示方法
/dev/XdYZ
/dev	表示的是一个设备目录
X	h  IDE磁盘
	s  SATA，SCSI，U盘
Y	a  第1块硬盘
	b  第2块硬盘
	c  第3块硬盘
	d  第4块硬盘
	......
Z	1-4表示1-4个主分区
	5表示第1个逻辑分区
	6表示第2个逻辑分区
	......
逻辑分区数量
	IDE	59个 （5~63）
	SATA	11个（5~15）
3，Linux使用文件系统
	Ext3
	SWAP	8G
	同时支持Windows的Fat分区，但是默认对NTFS不可写
swap功能：
	当有数据存放在物理内存里面，但是这些数据又不是常被CPU调用，那么这些不常用的程序会被丢到硬盘的swap交换分区当中，而将速度较快的物理内存空间释放出来给真正需要程序使用。
4，Linux目录结构
/	根分区，Linux文件系统的起点
/bin	普通用户使用命令
/sbin	管理员使用命令
/home	普通用户的家目录
/root	管理员的家目录
/boot	Linux启动所需文件存放目录（内核，grub引导程序）
/dev	设备文件，键盘鼠标
/proc	虚拟文件系统，（计算机内存信息，cpu...）不占用真实硬盘空间，是内存的映射

5，建议分区方案
/boot	100M
swap	2G ~ 8G
/	20G
/data	剩余所有空闲空间

三、RHEL5的基本操作
1，账户
windows	管理员 administrator
linux	管理员 root
2，图形模式与字符模式切换
ctrl+alt+Fn (n=1~6) 从图形切换到字符
alt+Fn		    从字符切换到其他模式
alt+F7		    回到图形模式
3，命令提示符
# 代表管理员
$ 代表普通用户
[登录用户@主机名 工作目录]#
4，基本命令
# uname -r			查询内核
# cat /etc/redhat-release	查询具体操作系统版本
# lsb_release -a		查询系统版本
# hostname			查看主机名
# hostname teacher.tarena.com  	设置主机名
# ifconfig eth0			查看第一块网卡信息
# ifconfig eth0 1.1.1.1         设置ip
# cat /proc/cpuinfo		查看CPU相关信息
# cat /proc/meminfo		查看内存相关信息
# free -m			查看内存和SWAP相关信息
# exit				退出当前环境
关机命令
# shutdown -h now	
# init 0
# halt -p	
# poweroff
# shutdown -h +15 "XXXXXXXXXXXXXXXXXX"
重启命令
# shutdown -r now
# reboot	
# init 6

补充：
Linux下文件颜色意义
蓝色    -->  目录
绿色    -->  可执行文件
红色    -->  压缩文件
浅蓝色  -->  链接文件
白色    -->  其他文件
黄色    -->  设备文件


# date	查看时间
# date MMDDhhmmYYYY   date 082917162013
# cal	查看日历信息
# cal 2013
# cal 09 2013
# bc	计算器
  scale=2	保留2位小数点