1、邮件的基本概念
MUA：邮件用户代理，客户端收发邮件的软件
MTA：邮件传输代理，服务器上的部署邮件服务器的软件
MDA：邮件投递代理，在邮件服务器上将邮件存放到相应的位置
MRA：邮件收取代理，为MUA读取邮件提供标准接口，主要使用POP3和IMAP协议
2、常用的MUA与MTA
MUA：Outlook、Mozilla Thunderbird、Foxmail
MTA：Sendmail、Postfix、Qmail、Exchange Server
3、邮件传递原理
发送邮件时：
	用户通过MUA将邮件投递到MTA
	MTA首先将邮件传给MDA
	MDA会根据邮件收件人的不同采取不同的方式处理
	 	收信人和发信人来自同一域：MDA将邮件存放到对应邮件存放地点
		收信人和发信人来自不同域：MDA将邮件还给MTA
	MTA通过DNS查询到收件人MTA的IP地址
	将邮件投递到收件人MTA
	收件人所在区域MTA将邮件投递到MDA
	MDA将邮件存放到对应邮件存放地点
接受邮件时：	
	用户通过MUA连接MRA
	MRA在邮件存放地点将邮件收取，并传递回MUA
4、邮件中继
MTA收到邮件有两种处理方式，其中一种是向外投递邮件，这个就是中继。
当MTA投递邮件时，邮件的收件人和发件人都不是MTA所在区域，这个叫第三方中继
5、邮件相关协议
SMTP：简单邮件传输协议	TCP	25
POP3：邮局协议版本3	TCP	110
POPs：提供加密的POP3	TCP	995
IMAP：交互邮件访问协议	TCP	143
IMAPs：提供加密的IMAP	TCP	993
6、SMTP认证
SASL简单身份验证和安全层
7、postfix主配置文件内容
/etc/postfix/main.cf
	myorigin 	指定发件人DNS后缀
	mydestination	指定postfix允许处理的邮件
# postfix check


----------
                                                  RHEL5.9_A Mail Server
-------Server1（VM1）---------（VM1） 
      192.168.10.254                     Win7 Client

实验要求：
	在server1上搭建DNS，能够解析mail.tarena.com到Mail Server
	在Mail Server上部署邮件服务器
	在Win7上安装Foxmail测试

Server1配置（配置DNS）
1、安装软件包
[root@server1 ~]# rpm -q bind bind-chroot caching-nameserver
package bind is not installed
package bind-chroot is not installed
package caching-nameserver is not installed
[root@server1 ~]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY- redhat-release 
[root@server1 ~]# yum -y install bind bind-chroot caching- nameserver
2、修改主配置文件
[root@server1 ~]# cd /var/named/chroot/etc/
[root@server1 etc]# cp -p named.caching-nameserver.conf named.conf
...
 15         listen-on port 53 { 192.168.10.254; };
...
 27         allow-query     { any; };
 28         allow-query-cache { any; };
...
 37         match-clients      { any; };
 38         match-destinations { any; };
...
 41 zone "tarena.com" IN {
 42         type master;
 43         file "tarena.com.zone";
 44         };
...
[root@server1 etc]# named-checkconf named.conf 
3、配置区域文件
[root@server1 etc]# cd /var/named/chroot/var/named/
[root@server1 named]# cp -p named.zero tarena.com.zone
[root@server1 named]# cat tarena.com.zone 
$TTL    86400
@               IN SOA  tarena.com.      root.tarena.com. (
                                        2013122401  ; serial (d.  adams)
                                        3H          ; refresh
                                        15M         ; retry
                                        1W          ; expiry
                                        1D )        ; minimum
        IN      NS      dns1.tarena.com.
        IN      MX   5  mail.tarena.com.
dns1    IN      A       192.168.10.254
mail    IN      A       192.168.10.10
[root@server1 named]# named-checkzone tarena.com tarena.com.zone 
zone tarena.com/IN: loaded serial 2013122401
OK
4、启动DNS
[root@server1 named]# service named restart
[root@server1 named]# chkconfig named on

Mail Server配置（配置SMTP服务器）
1、前提条件（测试DNS）
[root@mail ~]# cat /etc/sysconfig/network
NETWORKING=yes
NETWORKING_IPV6=yes
HOSTNAME=mail.tarena.com
GATEWAY=192.168.10.254
[root@mail ~]# hostname mail.tarena.com
[root@mail ~]# cat /etc/resolv.conf 
nameserver 192.168.10.254
nameserver 202.106.0.20
search tarena.com
[root@mail ~]# host -t mx tarena.com
tarena.com mail is handled by 5 mail.tarena.com.
[root@mail ~]# host mail.tarena.com
mail.tarena.com has address 192.168.10.10
2、安装Postfix
[root@mail ~]# netstat -tulnp | grep :25
tcp    0   0 127.0.0.1:25   0.0.0.0:*   LISTEN    4079/sendmail  
[root@mail ~]# service sendmail stop
[root@mail ~]# chkconfig sendmail off
[root@mail ~]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat- release
[root@mail ~]# yum -y install postfix
[root@mail ~]# chkconfig --add postfix
[root@mail ~]# chkconfig --list postfix
postfix    0:关闭  1:关闭  2:启用  3:启用  4:启用  5:启用  6:关闭
3、修改主配置文件
[root@mail ~]# cd /etc/postfix/
[root@mail postfix]# postconf -n > tmp.txt
[root@mail postfix]# mv main.cf main.cf.bak
[root@mail postfix]# mv tmp.txt main.cf
[root@mail postfix]# vim main.cf
...
  8 #inet_interfaces = localhost     //监听端口
 20 myhostname = mail.tarena.com     //邮件服务器主机名
 21 mydomain = tarena.com            //邮件服务器所在区域
 22 myorigin = $mydomain             //发件人DNS后缀            
 23 mydestination = $mydomain        //指定Postfix允许处理的邮件 
 24 home_mailbox = Maildir/	     //邮箱类型
 25 mynetworks = 192.168.10.0/24     //设置允许哪些客户端直接将需 要转发到外部区域的邮件提交给Postfix
4、检查语法启动服务
[root@mail postfix]# postfix check
[root@mail postfix]# postfix reload
[root@mail postfix]# netstat -tulnp | grep :25
tcp   0   0 0.0.0.0:25   0.0.0.0:*       LISTEN     6015/master    

测试：
[root@mail ~]# useradd yg
[root@mail ~]# useradd xln
[root@mail ~]# echo 123456 | passwd --stdin yg
[root@mail ~]# echo 123456 | passwd --stdin xln
[root@mail ~]# telnet mail.tarena.com 25
Trying 192.168.10.10...
Connected to mail.tarena.com (192.168.10.10).
Escape character is '^]'.
220 mail.tarena.com ESMTP Postfix
helo localhost			//宣告客户端
250 mail.tarena.com
mail from:yg@tarena.com		//邮件来自
250 2.1.0 Ok
rcpt to:xln@tarena.com		//邮件发往
250 2.1.5 Ok
data				//邮件正文
354 End data with <CR><LF>.<CR><LF>
subject Test mail!		//邮件主题
hello,byebye            
.				//邮件结束
250 2.0.0 Ok: queued as A967D324DE8
quit				//退出
221 2.0.0 Bye
Connection closed by foreign host.
[root@mail ~]# tail /var/log/maillog 
...
Dec 24 11:49:27 ser1 postfix/smtpd[14064]: connect from  ser1.tarena.com[192.168.10.10]
Dec 24 11:50:20 ser1 postfix/smtpd[14064]: A967D324DE8:  client=ser1.tarena.com[192.168.10.10]
Dec 24 11:51:09 ser1 postfix/cleanup[14083]: A967D324DE8:  message-id=<20131224035020.A967D324DE8@mail.tarena.com>
Dec 24 11:51:09 ser1 postfix/qmgr[13913]: A967D324DE8:  from=<yg@tarena.com>, size=367, nrcpt=1 (queue active)
Dec 24 11:51:09 ser1 postfix/local[14099]: A967D324DE8:  to=<xln@tarena.com>, relay=local, delay=63, delays=63/0.01/0/0.05,  dsn=2.0.0, status=sent (delivered to maildir)
Dec 24 11:51:09 ser1 postfix/qmgr[13913]: A967D324DE8: removed
Dec 24 11:51:14 ser1 postfix/smtpd[14064]: disconnect from  ser1.tarena.com[192.168.10.10]
[root@mail ~]# ls ~xln/Maildir/new/
1387857069.V802I3ec114M561364.mail.tarena.com
[root@mail ~]# cat  ~xln/Maildir/new/1387857069.V802I3ec114M561364.mail.tarena.com 
Return-Path: <yg@tarena.com>
X-Original-To: xln@tarena.com
Delivered-To: xln@tarena.com
Received: from localhost (ser1.tarena.com [192.168.10.10])
        by mail.tarena.com (Postfix) with SMTP id A967D324DE8
        for <xln@tarena.com>; Tue, 24 Dec 2013 11:50:06 +0800  (CST)
Message-Id: <20131224035020.A967D324DE8@mail.tarena.com>
Date: Tue, 24 Dec 2013 11:50:06 +0800 (CST)
From: yg@tarena.com
To: undisclosed-recipients:;

subject Test mail!
hello,byebye

Mail Server配置（配置POP服务器）
1、安装dovecot
[root@mail ~]# yum -y install dovecot
2、配置主配置文件
[root@mail ~]# vim /etc/dovecot.conf 
...
  85 ssl_disable = yes				//禁用SSL加密
...
 205    mail_location = maildir:~/Maildir	//设置邮箱路径
3、启动服务
[root@mail ~]# service dovecot restart
[root@mail ~]# chkconfig dovecot on
[root@mail ~]# netstat -tulnp | grep dovecot
tcp   0      0 :::110      :::*        LISTEN      16835/dovecot   
tcp   0      0 :::143      :::*        LISTEN      16835/dovecot  

测试：
[root@mail ~]# telnet mail.tarena.com 110
Trying 192.168.10.10...
Connected to mail.tarena.com (192.168.10.10).
Escape character is '^]'.
+OK Dovecot ready.
user xln		//输入账户
+OK
pass 123456		//输入密码
+OK Logged in.
list			//列出邮件
+OK 1 messages:	
1 458
.
retr 1			//查看邮件1
+OK 458 octets
Return-Path: <yg@tarena.com>
X-Original-To: xln@tarena.com
Delivered-To: xln@tarena.com
Received: from localhost (ser1.tarena.com [192.168.10.10])
        by mail.tarena.com (Postfix) with SMTP id A967D324DE8
        for <xln@tarena.com>; Tue, 24 Dec 2013 11:50:06 +0800  (CST)
Message-Id: <20131224035020.A967D324DE8@mail.tarena.com>
Date: Tue, 24 Dec 2013 11:50:06 +0800 (CST)
From: yg@tarena.com
To: undisclosed-recipients:;

subject Test mail!
hello,bybye
.
quit
+OK Logging out.
Connection closed by foreign host.

SMTP认证控制
1、启动saslauthd服务
[root@mail ~]# rpm -q cyrus-sasl
cyrus-sasl-2.1.22-7.el5_8.1
[root@mail ~]# cat /etc/sasl2/smtpd.conf
pwcheck_method: saslauthd
[root@mail ~]# service saslauthd start
[root@mail ~]# chkconfig saslauthd on
[root@mail ~]# testsaslauthd -u yg -p 123456 -s smtp	//检查 saslauthd服务
0: OK "Success."
2、调整postfix配置，启用认证
[root@localhost ~]# vim /etc/postfix/main.cf
...
 25 mynetworks = 127.0.0.1			//设置本地网络
 26 smtpd_sasl_auth_enable = yes		//启用SASL认证
 27 smtpd_sasl_security_options = noanonymous	//阻止匿名发信
 28 smtpd_recipient_restrictions =		//设置收件人过滤
 29  permit_mynetworks,
 30  permit_sasl_authenticated,
 31  reject_unauth_destination			//拒绝向未授权的目 标域发信
[root@localhost ~]# service postfix restart

3、测试
[root@mail ~]# printf "yg" | openssl base64
eWc=
[root@mail ~]# printf "123456" | openssl base64
MTIzNDU2
[root@mail ~]# telnet mail.tarena.com 25
Trying 192.168.10.10...
Connected to mail.tarena.com (192.168.10.10).
Escape character is '^]'.
220 mail.tarena.com ESMTP Postfix
mail from:yg@tarena.com
250 2.1.0 Ok
rcpt to:john@ibm.com.cn
554 5.7.1 <john@ibm.com.cn>: Relay access denied
quit
221 2.0.0 Bye
Connection closed by foreign host.
[root@mail ~]# !telnet
telnet mail.tarena.com 25
Trying 192.168.10.10...
Connected to mail.tarena.com (192.168.10.10).
Escape character is '^]'.
220 mail.tarena.com ESMTP Postfix
helo localhost
250 mail.tarena.com
auth login
334 VXNlcm5hbWU6
eWc=
334 UGFzc3dvcmQ6
MTIzNDU2
235 2.0.0 Authentication successful
mail from:yg@tarena.com  
250 2.1.0 Ok
rcpt to:john@ibm.com.cn
250 2.1.5 Ok
quit
221 2.0.0 Bye
Connection closed by foreign host.

邮件过滤
1、根据客户端地址过滤
[root@mail ~]# tail -n 2 /etc/postfix/access 
192.168.10.53   REJECT
192.168.10.50   OK
[root@mail ~]# postmap /etc/postfix/access 
[root@mail ~]# vim /etc/postfix/main.cf
...
 32 smtpd_client_restrictions = check_client_access  hash:/etc/postfix/access
[root@localhost ~]# postfix reload
做完这个实验请将main.cf 32行注释

2、根据发信人地址过滤
[root@mail ~]# cat /etc/postfix/sender_access
yg@tarena.com   REJECT
[root@mail ~]# postmap /etc/postfix/sender_access 
[root@mail ~]# vim /etc/postfix/main.cf
...
 33 smtpd_sender_restrictions =
 34  permit_mynetworks,		  //若从mynetworks网络访问则允许
 35  reject_sender_login_mismatch,//发件人与登录信息不符时拒绝
 36  reject_non_fqdn_sender,	  //拒绝不完整的发件域
 37  reject_unknown_sender_domain,	//拒绝未知的收件域
 38  check_sender_access hash:/etc/postfix/sender_access                                                  //指定策略库
[root@localhost ~]# service postfix restart
做完这个实验请将main.cf 33-38行注释



