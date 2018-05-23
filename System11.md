一、计划任务管理
1，at	一次性执行进程atd        /var/spool/at下
date	查看当前时间
atq	查询at任务
atrm	删除at任务
# at 10:05
# at 10:05 2013-12-20
# at 10pm december 14
# at now +5 [minutes|hours|days|weeks]

2，cron	周期性任务服务名crond	/var/spool/cron 
软件包
        vixie-cron 
        crontabs
crontab -e [ -u username]	编辑
        -l  	查看
        -r  	删除 
格式：
分 时 日 月 周	命令
*    表示该范围内的任意时间
,    表示间隔的多个不连续时间点
-    表示一个连续的时间范围
/    指定间隔的时间频率
3，计划任务授权
at任务控制
	/etc/at.allow、/etc/at.deny
crond任务控制
	/etc/cron.allow、/etc/cron.deny
如果allow文件存在，则仅允许指定的用户
否则检查deny文件，除指定用户外其余都允许
如果两个文件都不存在，则只允许root使用
4，cron计划中断与补救
anacron延时补救
/etc/init.d/anacron
/etc/anacrontab
1       65       cron.daily     run-parts /etc/cron.daily
1天    65分钟后
二、系统日志管理
1，日志文件的分类(日志保存在/var/log下)
内核及系统日志
由系统服务syslog统一进行管理
	/var/log/messages       内核及公共消息日志
	/var/log/cron	    计划任务日志
	/var/log/dmesg	    系统引导日志
	/var/log/maillog           邮件系统日志
	/var/log/secure	    记录与访问限制相关日志
用户日志
记录系统用户登录及退出系统的相关信息、二进制文件（不能用cat\more\less查看）
	/var/log/lastlog    最近的用户登录事件
	/var/log/wtmp	    用户登录、注销及系统开关机
	/var/log/btmp 	    失败的用户登陆事件
	/var/run/utmp	    当前登录的每个用户详细信息	
程序日志
由各种应用程序独立管理的日志文件，记录格式不统一
通用：用tail\less\grep等文本浏览、awk\sed等格式化过滤工具
专用：Webmin系统管理套件、Webalizer\AWstats等日志统计套件

2，用户日志分析
users	当前登陆的用户
w	查看当前登陆的用户
who	比w更加简洁  （pts/1  图形界面终端、：0 开机登录用户名）
last  -2	最近系统登陆情况
lastb  -2	最近系统登陆失败的情况
3，系统及内核日志格式
时间标签	   主机名    子系统名称    消息
4，syslogd管理日志
配置文件：/etc/syslog.conf
格式如下
服务类别.日志级别           日志消息发送位置 
5，日志消息的级别
0  EMERG（紧急）		会导致主机系统不可用的情况
1  ALERT（警告）		必须马上采取措施解决的问题
2  CRIT（严重）		比较严重的情况
3  ERR（错误）		运行出现错误
4  WARNING（提醒）	可能会影响系统功能的事件
5  NOTICE（注意）	                不会影响系统但值得注意
6  INFO（信息）		一般信息
7  DEBUG（调试）		程序或系统调试信息等

三、logrotate日志轮转
1，logrotate轮转
	减小日志大小，降低分析难度
	丢弃过期日志节省空间
	结合cron每天执行
2，软件包
	logrotate
3，主配置文件（daily, weekly, monthly, or yearly)
	/etc/logrotate.conf
	weekly  	轮转频率，默认每周
	rotate 4  	保留4个轮转备份
	create  	执行轮转后创建新文件
	#compress  	是否压缩日志
	include /etc/logrotate.d  	包含此目录下的配置
	/var/log/wtmp {  		启用轮转的日志文件
    		monthly  		每月轮转一次
		missingok  	丢失不提示
    		notifempty 	如果为空则不轮转
    		minsize 1M  	日志达到1MB才开始轮转
    		create 0664 root utmp  建新文件并设权限
    		rotate 1  		     只保留一个备份
		}	
logrotate	手工执行轮转
	-v	启动显示模式
	-f	强制rotate
/var/lib/logrotate.status






########################测试题############################
二、系统日志管理
1，开启二台主机，其中一台作为日志服务器，另一台作为客户端
2，将客户端的cron日志除了正常保存本地外让日志服务器也能接收

三、logrotate日志轮转
1，把当前Linux系统产生的日志额外都保存到/var/log/admin.log里面去
2，设置/var/log/admin.log文件每天轮转一次，该文件若大于10k时则主动进行轮转。保存3个备份文件并且需要压缩



一、查看及测试网络
1，ifconfig 	查看活动接口信息
   ifconfig -a	查看所有接口信息（包括非活动）
   ifconfig eth0	查看指定eth0接口信息
2，hostname	查看主机名
3，route -n	查看网关(UG)
4，arp -n		查看arp缓存表 
5，netstat -nr	= route -n	查看网关
   netstat -ln | grep :端口	查看该端口是否开启
   netstat -tulnp | grep 服务名	查看该服务开启那些端口
   netstat -anptu | grep ESTABL	查看已与本机建立的TCP连接
6，ping	-c 包个数	测试网络连通性
7，traceroute = tracert	路由跟踪
8，host 域名 	 	DNS解析
   nslookup 域名
   	server		查看当前主机的dns地址

二、配置网络地址
使用命令调整网络参数（临时）
①设置ip
   ifconfig eth0 ip	设置ip
   ifdown eth0	禁用eth0
   ifup eth0	启用eth0
   ifconfig eth0:0 ip	设置虚拟接口ip
②添加删除默认网关（网关必须和当前主机在同一网段）
   route add  default  gw  网关
   route del  default  gw   网关
   添加删除指定路由
   route add  -net  网段地址/子网掩码  gw  网关
   route del  -net  网段地址/子网掩码
③hostname 主机名
④arp -s ip mac		静态绑定
   arp -d ip		删除指定arp记录


开启路由转发
echo "1" > /proc/sys/net/ipv4/ip_forward  (临时的)
或修改配置文件  /etc/sysctl.conf 
# vim /etc/sysctl.conf 
net.ipv4.ip_forward = 1
# sysctl -p   （立即生效）



通过配置文件修改网络参数（永久）
需要重启network服务后生效
①设置ip
#vim /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0	             设备名
ONBOOT=yes	             启动是否加载
BOOTPROTO=static	  手动 dhcp则为自动获取，下面的ip参数就不用设置
IPADDR=192.168.4.1             ip地址
NETMASK=255.255.255.0      子网掩码
MACADDR=xx:xx:xx:xx:xx:xx   修改mac地址
PEERDNS=no	              禁止dhcp获取时修改resolv.conf文件
#ifdown  eth0  	停用eth0
#ifup  eth0    	激活eth0
②修改主机名与网关
#vim /etc/sysconfig/network
HOSTNAME=XXX
GATEWAY=192.168.4.254	网关
③设置域名ip对应关系
#vim /etc/hosts		ip地址与主机名对应关系
格式：
ip	FQDN	主机名
注：计算机改名后将ip与主机名的对应关系写到hosts文件中，否则重启过程非常慢（将进行主机名解析）

④指定DNS地址（立即生效）
#vim /etc/resolv.conf	设置dns，最多3条
search xxx.com
nameserver 202.106.0.20

三、内核与模块管理
1，lsmod 		列出已加载的所有模块
2，modinfo	查看指定模块的信息
3，modprobe	智能检测及加载模块
	 -r 	卸载模块  =rmmods
 insmod 绝对路径

四、内核与模块管理
uname -r

1，使用rpm升级内核
    1)安装软件包
       [root@student1 Desktop]# rpm -ivh kernel-2.6.32.61-5.x86_64.rpm
    2)创建initrd-
       [root@student1 Desktop]# mkinitrd /boot/initrd-2.6.32.61.img 2.6.32.61
    3)
      [root@student1 Desktop]# vim /boot/grub/grub.conf 
    4)重启
2，查看是否加载ext4模块
lsmod | grep ext4

3，查看ext4模块详细信息
modinfo ext4

4，加载ext4模块
modprobe ext4

5，移除ext4模块
modprobe -r ext4