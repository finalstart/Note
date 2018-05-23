一、光盘文件使用
1，RHEL5（x86_64）光盘结构
Cluster         //集群二进制包
ClusterStorage  //集群文件系统二进制包
Server          //核心服务器的二进制包产品
VT              //虚拟化二进制包
image	        //引导和驱动程序磁盘映像
isolinux	//图像引导文件
2，挂载光盘（放入光盘）
/dev/cdrom = /dev/hdc		光盘驱动器设备文件 
#umount /dev/cdrom		卸载光盘使用
#mount /dev/cdrom /media/	挂载光盘使用
#ls /media/			查看光盘的内容

二、RPM软件包管理
1，RPM数据库文件
/var/lib/rpm
2，查询已安装的RPM软件包信息
rpm -q  软件包名称		查询指定包是否安装
rpm -qa				查询系统已经安装所有的软件包
rpm -qa | grep 软件包名称	查询当前系统安装了哪些与软件包名称相关的包
rpm -qi 软件包名称		查询已安装软件包的详细信息
rpm -ql 软件包名称  		查询已安装软件包安装到什么地方去了
rpm -qc 软件包名称		查询软件生成的配置文件
3，查询某个目录或者文件是由哪个RPM包产生的
rpm -qf 文件的绝对路径		查询该文件由哪个包产生
4，查询待安装的RPM安装文件(先将rpm包传到/root/Desktop下)
rpm -qpi 完整软件包名称		查询未安装软件的详细信息 
rpm -qpl 完整软件包名称	查询未安装软件要安装的文件路径
5，验证已安装的软件包
rpm -V	软件名
    -Vf 文件路径
    -Va	列出系统中在RPM安装后改动过的所有文件
	S：文件大小
	M：权限或类型
	5：MD5校验和
	D：设备编号
	L：链接数
	U：用户
	G：组
	T：时间
6，导入官方公钥
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release 

7，安装升级RPM
rpm -i		安装
    -U  	升级  
    -F		升级（老版本未装不安装新版本）
    -v  	显示细节信息
    -h  	以#显示安装进度
    --force	强制安装
8，卸载
rpm -e 软件包名称

9，依赖关系


二、配置YUM库及更新操作
1，yum概述
	基于RPM包构建的软件更新机制，自动解决软件依赖关系

2,YUM仓库格式
	本地：file://
	网络：ftp://或http://
3,YUM仓库配置文件
	/etc/yum.repos.d/*.repo
4，搭建本地YUM仓库
①配置YUM源
放入5.9光盘，并挂载到/media
[root@localhost ~]# umount /dev/cdrom
[root@localhost ~]# ls /media/
[root@localhost ~]# mount /dev/cdrom /media/
[root@localhost ~]# ls /media/
②配置YUM客户端
[root@localhost ~]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# cp rhel-debuginfo.repo rhel5.9.repo
[root@localhost yum.repos.d]# cat rhel5.9.repo 
[rhel5.9-Server]
name=Red Hat Linux 5.9
baseurl=file:///media/Server
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
③测试
[root@localhost ~]# yum clean all
[root@localhost ~]# yum list | wc -l
3347

5,yum客户端操作
	yum
		list		查看软件包列表
		info		查看软件包的说明信息
		install		安装指定软件包
		update		升级指定软件包
		remove		卸载指定软件包
		--nogpgcheck	不验证gpg签名
6，清空本地yum缓存(/var/cache/yum)
	yum clean all
7，软件组基本操作
	yum 
		grouplist	查看软件组列表
		groupinfo	查看软件组说明信息
		groupinstall	安装指定软件组
		groupupdate	升级软件组
		groupremove	卸载软件组


########################测试题############################
一、RPM软件包管理
1，rpm操作
                查询是否安装firefox和setup
[root@localhost ~]# rpm -q firefox
[root@localhost ~]# rpm -q setup

                查询是否安装ftp相关的软件包，查询安装软件中以vim开头的软件
[root@localhost ~]# rpm -qa | grep ftp
[root@localhost ~]# rpm -qa | grep ^vim

                查询所有已安装的软件包
[root@localhost ~]# rpm -qa

                查询firefox的详细信息
[root@localhost ~]# rpm -qi firefox

                查询firefox安装到什么地方了
[root@localhost ~]# rpm -ql firefox

                查询setup安装到/etc下的文件
[root@localhost ~]# rpm -qc setup

                查询/etc/passwd是那个软件包产生的
[root@localhost ~]# rpm -qf /etc/passwd

                挂载5.9DVD光盘到/media下
[root@localhost ~]# mount /dev/cdrom /media

                查询dvd光盘中vsftpd这个软件包的详细信息
[root@localhost ~]# rpm -q vsftpd
package vsftpd is not installed
[root@localhost ~]# find /media/Server | grep vsftpd
/media/Server/vsftpd-2.0.5-28.el5.x86_64.rpm
[root@localhost ~]# rpm -qpi vsftpd-2.0.5-28.el5.x86_64.rpm

                查询dvd光盘中vsftpd这个软件包安装的话会安装到什么地方，可执行程序会安装到哪里去
[root@localhost Server]# rpm -qpl vsftpd-2.0.5-28.el5.x86_64.rpm 
[root@localhost Server]# rpm -qpc vsftpd-2.0.5-28.el5.x86_64.rpm 

                查询bash这个软件包安装的文件是否被改动
[root@localhost Server]# rpm -V bash

                查询/etc/passwd这个文件有没改动
[root@localhost Server]# rpm -Vf /etc/passwd

                列出系统中在RPM安装后改动过的所有文件
[root@localhost Server]# rpm -Va

                导入RPM公钥文件
[root@localhost Server]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release 

                测试vsftpd是否可以安装
                以#号显示安装进度并显示细节信息安装vsftpd
[root@localhost Server]# rpm -ivh vsftpd-2.0.5-28.el5.x86_64.rpm 

                查找vi、vim的位置，然后删除vi，vim程序，用rpm重装进行恢复
[root@localhost Server]# which vi vim
[root@localhost Server]# rm /bin/vi
[root@localhost Server]# rm /usr/bin/vim
[root@localhost Server]# rpm -qf /bin/vi /usr/bin/vim
[root@localhost Server]# rpm -ivh vim-minimal-7.0.109-7.2.el5.x86_64.rpm vim-enhanced-7.0.109-7.2.el5.x86_64.rpm --force

                卸载vsftpd
[root@localhost Server]# rpm -e vsftpd

二、配置YUM库及更新操作
[root@localhost Server]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# vim rh-server.repo

1，yum操作
                安装gcc
[root@localhost yum.repos.d]# yum -y install gcc

                卸载vim-common
[root@localhost yum.repos.d]# yum remove vim-common

                安装vim-enhanced
[root@localhost yum.repos.d]# yum -y install vim-enhanced

                查看软件包列表
[root@localhost Server]# yum list

                查看vsftpd的说明信息
[root@localhost yum.repos.d]# yum info vsftpd


                安装vsftpd      
[root@localhost yum.repos.d]# yum -y install vsftpd
  
                卸载vsftpd
[root@localhost yum.repos.d]# yum remove vsftpd

                查看软件组列表
[root@localhost yum.repos.d]# yum grouplist
                查看软件组DNS名称服务器的说明信息
[root@localhost Server]# yum groupinfo "DNS Name Server"

                安装DNS名称服务器软件组
[root@localhost Server]# yum groupinstall "DNS Name Server"

                卸载DNS名称服务器软件组
[root@localhost Server]# yum groupremove "DNS Name Server"

YUM配置FTP源。
linux和windows server 2008 R2 可以通信。
用windows Server 2008 R2 以光盘文件做个FTP。
将FTP的路径设为： E:/yum （由于光盘目录的名字太长，我改了为yum）

在linux下创建yum的源文件--- /etc/yum.repos.d/rhel-ftp.repo
[rhel-ftp]		//注意，这里最好不要存在空格，特殊符号等等。
name= This is ftp yum source		//注解，可以随便写。
baseurl= ftp://192.168.1.1/Server 	//这个地址根据自己的FTP的路径写，情况不一样，路径不一样。
enabled=1			//是否启用这个容器
gpgcheck=1		//是否使用公钥验证，咱们只是测试，可以不要。若不要，则改为0，当然，这里为0时，下边的公钥路径也可以不写。
gpgkey= file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release		//公钥的路径
