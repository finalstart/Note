一、DHCP搭建与维护
1，DHCP基本概念
	dynamic host configuration protocol 动态主机配置协议
2，工作原理
	dhcp discover
	dhcp offer
	dhcp request
	dhcp ack
3，提供参数
	ip netmask gateway dns broadcast
4，租约
	50%   --- dhcp request
	87.5% --- dhcp discover
5，端口
	udp 67  server
	udp 68  client
6，配置dhcp服务器
条件：
DHCP SERVER 必须有个固定ip
①，/etc/dhcpd.conf 主配置文件详解
subnet 		申明作用域
range  		地址池
host   		绑定主机

default-lease-time  （默认租约期）
max-lease-time      （最大租约期）	        
fixed-address        (为主机分配固定IP，只能用在host声明)	
hardware   ethernet  (指定主机物理地址，只能用在host声明)	
option  routers        （指定网关地址）
option  domain-name-servers    (指定DNS服务器地址)

②,启动
# service dhcpd configtest	测试dhcp服务器
# service dhcpd start/stop/restart/status/reload
# chkconfig --level 35 dhcpd on
# netstat -ln | grep 67
③,租约文件
/var/lib/dhcpd/dhcpd.leases
④,多网卡接口
/etc/sysconfig/dhcpd	提供dhcp服务的网络接口
7,客户端的配置
windows
	ipconfig /release
	ipconfig /renew
linux
# dhclient
	-d 指定接口
	-r 释放ip
# vim /etc/sysconfig/network-scripts/ifcfg-eth0
BOOTPROTO=dhcp   		//修改为dhcp
# service network restart
8、开启路由转发功能
# vim /etc/sysctl.conf 
...
  7 net.ipv4.ip_forward = 1
...
# sysctl -p

二、NFS共享服务
1，软件包
	nfs-utils	
	portmap	rpc  tcp/udp 111
2，主配置文件
	/etc/exports
格式：
共享目录	 客户机地址(参数,参数)
客户端地址
	IP地址：192.168.4.20
	网段地址：172.0.0.0/24 或 172.0.0.*
	所有主机：*
	单个域：*.tarena.com 
	主机名：pc110.tarena.com
参数
	rw、ro：可读可写、只读
	sync、async：同步写、异步写入
	no_root_squash：保留来自客户端的root权限
	all_squash：客户端权限都降为nfsnobody
3，启动nfs
	/etc/init.d/portmap restart
	/etc/init.d/nfs restart
4，showmount	
	-e	查看NFS共享列表
	-a	检查NFS使用情况
5，rpcinfo -p ip	查看RPC注册端口

三、NTP网络时间服务
1,NTP Network Time Protocol   网络时间协议
服务端工具：ntpd
客户端工具：ntpdate
UDP 123
2,主配置文件
/etc/ntp.conf
restrict	    权限控制
      kod	         开启阻止Kiss of Death包攻击
      nomodify    客户端不能更改ntp服务器的时间参数，但是可以校对时间
      notrap        不提供trap远程事件登陆
      nopeer        不予其他同一层的NTP服务器同步时间
     noquery       不提供ntp服务	
server	    设定上层NTP服务器	
3,端口
	udp 123
4,ntpdate同步时间
	ntpdate 服务器ip
5，与crond任务配合使用

## DHCP
RHEL5.9_A  DHCP Server	
-------Server1（VM1）---------（VM1） RHEL5.9_B  Client
      192.168.10.254                                   Win7  	 Client

实验需求：
公司要求将闲置的一台Linux 主机配置为DHCP 服务器，以便为局域网内员工的办公用机提供自动分配IP地址的服务，以提高网络管理和维护的效率。需要满足的基本要求如下所述。
1．为192.168.10.0/24网段的客户机自动配置网络参数。
	用来给客户机自动分配的IP地址范围是：192.168.10.50-	192.168.10.100、192.168.10.120-192.168.10.200。
	客户机的默认网关地址设为192.168.10.254。
	客户机所使用的DNS服务器设为192.168.10.254、202.106.0.20	，默认搜索域后缀为tarena.com。
	将默认租约时间设为8小时，最大租约时间24小时
2．为打印服务器分配保留地址（Win7）
	这台打印机每次开启电源后获得的IP地址都应该是192.168.10.8。
3．验证DHCP服务器的IP分配情况、客户机的租约信息

DHCP服务器的搭建
1、设置RHEL5.9_A的ip
[root@ser1 ~]# ifconfig eth0 | grep "inet addr"
          inet addr:192.168.10.10  Bcast:192.168.10.255  Mask:255.255.255.0
2、安装软件包
[root@ser1 ~]# rpm -q dhcp
package dhcp is not installed
[root@ser1 ~]# cd /etc/yum.repos.d/
[root@ser1 yum.repos.d]# cp -p rhel-debuginfo.repo server.repo
[root@ser1 yum.repos.d]# cat server.repo 
[server]
name=Red Hat Enterprise Linux Server 
baseurl=ftp://192.168.10.254/pub/OS/RedHat/5.9/Server/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
[root@ser1 yum.repos.d]# yum clean all
[root@ser1 yum.repos.d]# yum -y install dhcp
3、修改主配置文件
[root@ser1 ~]# cat /etc/dhcpd.conf 
ddns-update-style interim;
subnet 192.168.10.0 netmask 255.255.255.0 {
        option routers                  192.168.10.254;
        option subnet-mask              255.255.255.0;
        option domain-name              "tarena.com";
        option domain-name-servers      192.168.10.254,202.106.0.20;
        range dynamic-bootp 192.168.10.50 192.168.10.100;
        range dynamic-bootp 192.168.10.120 192.168.10.200;
        default-lease-time 28800;
        max-lease-time 86400;
        host win7 {
                hardware ethernet 00:0C:29:6C:D1:85;
                fixed-address 192.168.10.8;
        }
}
4、启动服务
[root@ser1 ~]# service dhcpd restart
关闭 dhcpd：[确定]
启动 dhcpd：[确定]
[root@ser1 ~]# chkconfig dhcpd on
[root@ser1 ~]# netstat -ln | grep :67
udp        0      0 0.0.0.0:67                  0.0.0.0:*

客户端的测试
Linux：
[root@ser2 ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0
# Intel Corporation 82545EM Gigabit Ethernet Controller (Copper)
DEVICE=eth0
BOOTPROTO=dhcp
ONBOOT=yes
HWADDR=00:0c:29:01:94:66
[root@ser2 ~]# service network restart
[root@ser2 ~]# ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 00:0C:29:01:94:66  
          inet addr:192.168.10.200  Bcast:192.168.10.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe01:9466/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:54 errors:0 dropped:0 overruns:0 frame:0
          TX packets:71 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:7534 (7.3 KiB)  TX bytes:12356 (12.0 KiB)
Windows：（首先要设置自动获取）
	ipconfig /release	释放ip
	ipconfig /renew	重新获取ip

租约文件
服务器端： /var/lib/dhcpd/dhcpd.leases
客户端：    /var/lib/dhclient/dhclient.leases 

## NFS
		              	       RHEL5.9_A NFS Server
-------Server1（VM1）---------（VM1） 
          192.168.10.254                     RHEL5.9_B  NFS Client
实验需求：
将/root 共享给192.168.10.20，可写、同步，允许客户机以root权限访问
将/usr/src 共享给192.168.10.0/24网段，可写、异步，访问权限均降为nfsnobody用户

NFS服务器的配置
1、安装软件包
[root@ser1 ~]# rpm -q nfs-utils portmap
nfs-utils-1.0.9-66.el5
portmap-4.0-65.2.2.1
2、修改主配置文件
[root@ser1 ~]# cat /etc/exports
/root   192.168.10.20(rw,sync,no_root_squash)
/usr/src        192.168.10.0/24(rw,async,all_squash)
3、启动服务
[root@ser1 ~]# service portmap restart
[root@ser1 ~]# service nfs restart
[root@ser1 ~]# chkconfig portmap on
[root@ser1 ~]# chkconfig nfs on
4、设置目录权限
[root@ser1 ~]# setfacl -m u:nfsnobody:rwx /usr/src/

客户端测试：
[root@ser2 ~]# showmount -e 192.168.10.10
Export list for 192.168.10.10:
/root    192.168.10.20
/usr/src 192.168.10.0/24
[root@ser2 ~]# mkdir -p /data/{src,root}
[root@ser2 ~]# mount -t nfs 192.168.10.10:/root/ /data/root/
[root@ser2 ~]# mount -t nfs 192.168.10.10:/usr/src/ /data/src/
[root@ser2 ~]# touch /data/root/file1.txt
[root@ser2 ~]# touch /data/src/file1.txt
[root@ser2 ~]# ls -l /data/{root,src}/file1.txt
-rw-r--r-- 1 root      root      0 11-19 17:36 /data/root/file1.txt
-rw-r--r-- 1 nfsnobody nfsnobody 0 11-19 17:36 /data/src/file1.txt



公司的某个软件项目处于内部测试期间，有2台应用服务器短期内需要共享使用不少于500GB的磁盘空间，
要求当软件在执行读写操作时能像访问本机的目录一样，相关条件和需求如下所述。
1．将服务器192.168.10.10的/ptest目录作为共享，此目录已经挂载一个容量为500GB的逻辑卷。
2．/ptest目录仅允许指定的2台服务器（192.168.10.20、192.168.10.21）访问。
3．当从192.168.10.20挂载/ptest共享时，保留root的身份及完整权限
4．当从192.168.10.21挂载/ptest共享时，只有读取权限，不可写入，所有用户均视为nfsnobody对待。



