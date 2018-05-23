一、Samba基本共享
1,概念
	实现windows与linux系统之间的共享
	支持打印服务
	使用SMB/CIFS协议
2,主要软件包
	samba-common
	samba
	samba-client
3,主要程序
	smbd 提供对文件/打印资源的共享访问 TCP 139 TCP 445 
	nmbd 提供netbios主机名称解析          UDP 137 UDP 138
4,系统服务脚本
	/etc/init.d/smb
5,主配置文件
	/etc/samba/smb.conf
6,配置文件检查工具
	testparm
7,smb.conf配置解析
	[global]	samba全局设置，对整个samba有效
	[homes]		设置了用户共享目录属性，不包括全局
	[printers]	打印机共享
	[myshare]	自定义共享目录设置
samba安全级别
security         
	share	匿名共享
        user    要用户名和密码，由samba服务器检查（默认）
	server	由指定服务器认证
	domain	域控制器来验证
常见共享目录配置项	
path=/共享路径
writable=yes	   是否可写
browseable=no	   是否可浏览（默认可以）
read only=yes	   是否只读
public=yes	   是否允许guest访问 = guest ok=yes
guest only=yes	   是否只允许guest访问
comment=public	   描述，说明
read list=tom,@gr       只读用户访问列表
write list=tom,@gr      读写用户访问列表
valid users=tom,@gr   允许使用服务的用户列表
directory mask=0755   创建目录的默认权限
create mask=0644	    创建文件的默认权限
8,服务优化     		
deadtime = 10           10分钟不对服务器操作就中断连接
client code page = 936  客户端支持中文
9,客户端访问
windows
UNC路径	\\服务器地址\共享名
net use * /del （windows客户端删除连接信息）
Linux
#smbclient -L   服务器 显示指定samba服务器共享列表 
#mount -t cifs -o username=administrator //ip/public/mnt/smbfs
#smbstatus      显示当前服务器的连接状况信息(server上面)	
#umount /mnt/smbfs	

二、Samba共享的访问控制
1,samba帐号：
	samba账号必须系统真实存在
	samba账号密码独立的，和系统密码没关系
#pdbedit -a smb01 添加密码
	 -L smb01 查看共享账户信息
	 -x smb01 删除帐号
#smbpasswd smb01  重设密码
2,别名映射
	username map = /etc/samba/smbusers
   /etc/samba/smbusers格式
	共享账户 = 别名1 别名2 ..
3,访问地址限制
	hosts allow = 客户端地址
	hosts deny = 客户端地址 
客户端地址格式
	以空格分隔多个地址
	主机名或IP地址
	网络地址 192.168.10. 或者192.168.10.0/255.255.255.0

三、Vsftpd基础配置
1,FTP概述
File Transfer Protocol文件传输协议
2,传输模式
	主动模式：由服务器主动连接客户端建立数据链路
	被动模式：FTP服务器等待客户端建立数据链路
3,使用端口
	21 用于与客户机建立命令链路
	20 在主动模式下服务器使用20向客户机建立数据链路

4,配置vsftpd
软件包名
	vsftpd
主配置文件
/etc/vsftpd/vsftpd.conf     vsftpd服务器的主配置文件
/etc/vsftpd/ftpusers	        黑名单
/etc/vsftpd/user_list	        白/黑名单(缺省黑名单)
userlist_deny	        是否禁用user_list中的用户
当userlist_deny=YES	        被禁止(黑名单)
当userlist_deny=NO	        被允许(白名单)
/var/ftp/	默认共享出来的目录,权限不能更改   
匿名用户登陆用
	ftp	
	anonymous
5，参数
download_enable	  	是否允许下载
userlist_enable	  	是否启用user_list列表文件
max_clients	  	限制并发的客户端个数
max_per_ip	 	限制每个客户机IP的并发连接数
anonymous_enable	  	是否启用匿名访问
anon_root		  	匿名FTP的根目录
anon_upload_enable	  	是否允许上传文件
anon_mkdir_write_enable	是否允许建目录
anon_other_write_enable	其他写入控制
anon_max_rate		最大传输速度（字节/秒）
local_enable		是否启用本地用户
local_root			本地用户的FTP根目录
chroot_local_user		是否禁锢在主目录
local_max_rate		最大传输速率（字节/秒）
	
6，访问
ftp://服务器ip地址
   常用ftp命令
?        !        lcd        pwd	get	put	mget	mput

## samba
		              	      RHEL5.9_A Samba Server
-------Server1（VM1）---------（VM1） RHEL5.9_B Samba Client
      192.168.10.254                  真实机Win7 Client

实验要求：
	将目录 /usr/src 共享给所有人
	共享名设为 tools
	允许所有人访问、无需密码验证
	访问权限为只读

配置samba服务器
1、安装软件包
[root@ser1 ~]# rpm -qa | grep samba
[root@ser1 ~]# yum -y install samba samba-client samba-common
2、修改主配置文件
[root@ser1 ~]# vim /etc/samba/smb.conf 
...
 74         workgroup = Tarena
 75         server string = Win File Ser
...
 89         log file = /var/log/samba/%m.log
...
 91         max log size = 50
...
101         security = share
...
289 [tools]
290         comment = Tools Public
291         path = /usr/src
292         public = yes
293         read only = yes 
3、启动服务
[root@ser1 ~]# testparm 
[root@ser1 ~]# service smb restart
[root@ser1 ~]# chkconfig smb on
[root@ser1 ~]# netstat -anptu | grep mbd
tcp        0      0 0.0.0.0:139       0.0.0.0:*      LISTEN      7008/smbd         
tcp        0      0 0.0.0.0:445       0.0.0.0:*      LISTEN      7008/smbd         
udp        0      0 192.168.10.10:137       0.0.0.0:*           7011/nmbd        
udp        0      0 0.0.0.0:137                  0.0.0.0:*           7011/nmbd        
udp        0      0 192.168.10.10:138       0.0.0.0:*           7011/nmbd        
udp        0      0 0.0.0.0:138                 0.0.0.0:*            7011/nmbd           
客户端测试：
Windown:
	UNC路径 \\192.168.10.10
Linux:
[root@ser2 ~]# yum -y install samba-client
[root@ser2 ~]# smbclient -L 192.168.10.10
[root@ser2 ~]# smbclient //192.168.10.10/tools
[root@ser2 ~]# mkdir -p /data/smb
[root@ser2 ~]# mount //192.168.10.10/tools /data/smb/
[root@ser2 ~]# grep smb /etc/fstab 
//192.168.10.10/tools   /data/smb    cifs   defaults  0  0

	


Samba用户验证
实验要求：
	修改原有的 [tools] 匿名共享设置
	不再允许所有人访问
	只允许nick读取、hunter写入
	拒绝其他用户或匿名访问
1、新建相应账户与samba密码
[root@ser1 ~]# useradd nick
[root@ser1 ~]# useradd hunter
[root@ser1 ~]# echo "123456" | passwd --stdin nick
Changing password for user nick.
passwd: all authentication tokens updated successfully.
[root@ser1 ~]# echo "123456" | passwd --stdin hunter
Changing password for user hunter.
passwd: all authentication tokens updated successfully.
[root@ser1 ~]# pdbedit -a nick
[root@ser1 ~]# pdbedit -a hunter
2、修改主配置文件
[root@ser1 ~]# vim /etc/samba/smb.conf 
...
101         security = user
...
289 [tools]
290         comment = Tools Public
291         path = /usr/src
292         public = no
293         valid users = nick,hunter
294         write list = hunter
295         read only = yes
296         directory mask = 0755
297         create mask = 0644
...
[root@ser1 samba]# setfacl -m u:hunter:rwx /usr/src/
3、启动服务
[root@ser1 ~]# service smb restart
关闭 SMB 服务：                                             [确定]
关闭 NMB 服务：                                            [确定]
启动 SMB 服务：                                             [确定]
启动 NMB 服务：                                            [确定]

客户端测试
[root@ser2 ~]# smbclient -U nick //192.168.10.10/tools
[root@ser2 ~]# mount -o username=nick //192.168.10.10/tools /data/smb


账户别名
把系统帐户nick设置别名为jack
[root@ser1 ~]# vim /etc/samba/smbusers
# Unix_name = SMB_name1 SMB_name2 ...
root = administrator admin
nobody = guest pcguest smbguest
nick = jack
[root@ser1 ~]# vim /etc/samba/smb.conf 
...
 76         username map = /etc/samba/smbusers
...

## vsftp
		              	               RHEL5.9_A FTP Server
-------Server1（VM1）---------（VM1） RHEL5.9_B FTP Client
       192.168.10.254                                真实机Win7 Client

实验要求：
	配置可匿名上传FTP服务

服务器的搭建
1、安装软件包
[root@ser1 ~]# yum -y install vsftpd
2、修改主配置文件
[root@ser1 ~]# vim /etc/vsftpd/vsftpd.conf 
...
 27 anon_upload_enable=YES
...
 31 anon_mkdir_write_enable=YES
 32 anon_other_write_enable=YES
...
[root@ser1 ~]# setfacl -m u:ftp:rwx /var/ftp/pub/
3、启动服务
[root@ser1 ~]# service vsftpd restart
[root@ser1 ~]# chkconfig vsftpd on

客户端测试
ftp://192.168.10.10
ftp 192.168.10.10

实验要求：
	配置本地账户
	验证黑白名单
	
服务器的搭建
1、创建实验测试账户
[root@ser1 ~]# useradd lily
[root@ser1 ~]# useradd john
[root@ser1 ~]# useradd kaka
[root@ser1 ~]# echo 123456 | passwd --stdin lily
[root@ser1 ~]# echo 123456 | passwd --stdin john
[root@ser1 ~]# echo 123456 | passwd --stdin kaka
2、禁止匿名账户登录
[root@ser1 ~]# vim /etc/vsftpd/vsftpd.conf 
...
 12 anonymous_enable=NO
...
[root@ser1 ~]# service vsftpd restart
分别验证上面3个账户能否登录
3、验证黑名单
[root@ser1 ~]# grep lily /etc/vsftpd/ftpusers 
lily
测试：用lily登录
4、验证黑白名单
[root@ser1 ~]# grep john /etc/vsftpd/user_list 
john
测试：用john登录
[root@ser1 ~]# vim /etc/vsftpd/vsftpd.conf 
...
120 userlist_deny=NO
...
[root@ser1 ~]# service vsftpd restart
测试：lily john kaka 
who能登陆？who不能登录？why?

实验要求：
	禁锢普通账户在自己的宿主目录中    chroot_local_user=YES
	设置普通账户登录访问/data/ftp  local_root=/data/ftp
	限制普通账户下载速度150KB/s    local_max_rate=150000

1、修改主配置文件
[root@ser1 ~]# vim /etc/vsftpd/vsftpd.conf 
...
121 chroot_local_user=YES
...
[root@ser1 ~]# mkdir -p /data/ftp
[root@ser1 ~]# vim /etc/vsftpd/vsftpd.conf 
...
122 local_root=/data/ftp
123 local_max_rate=150000
...
[root@ser1 ~]# dd if=/dev/zero of=/data/ftp/test.db bs=10M count=100
100+0 records in
100+0 records out
1048576000 bytes (1.0 GB) copied, 14.5174 seconds, 72.2 MB/s
[root@ser1 ~]# service vsftpd restart
[root@ser2 ~]# wget ftp://john:123456@192.168.10.10/test.db
