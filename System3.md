一、文件内容操作（/etc/passwd）
1，cat	查看文件内容比较少的
2，more	可以分页显示
3，less	比more更全面
4，head	默认显示文件头10行
	-n 数字 	显示头n行
5，tail	默认显示文件尾10行
	-n 数字	显示尾n行
	-f	实时查看
6，wc	统计
7,grep	输出包含指定字符串的行
	-i	忽略大小写
	-v	取反
	^root	以root开头
	root$	以root结尾
	^$	匹配空行
	-E	查找多个关键字，匹配其中任意一个都输出
8，dmesg	 	查看启动信息
补充：
 |  管道 将前一个命令的输出结果交给后一个命令作为输入
 >  写入（清空之前文件内容，写入新内容）输出重定向
 >> 追加
# echo xxxxxx 原样在终端输出
二、压缩和归档
1，gzip	
gzip 文件名		压缩
gzip -d 文件名.gz	解压缩
2，bzip2
bzip2 文件名		压缩
bzip2 -d 文件名.bz2	解压缩
3，zip
zip 文件名.zip 文件名	压缩
unzip 文件名.zip		解压缩
     -d	指定解压位置
4，tar 打包
	-c	创建tar包
	-z	调用gzip
	-j	调用bzip2
	-x	解包
	-t	查看
	-C	指定解压位置
	-f	使用归档文件
	--remove	打完包后删除原文件
tar -zcf xxx.tar.gz xxx		调用gzip压缩
tar -jcf xxx.tar.bz2 xxx	调用bzip2压缩

tar -ztf xxx.tar.gz		查看xxx.tar.gz里面文件
tar -jtf xxx.tar.bz2		查看xxx.tar.bz2里面文件

tar -zxf xxx.tar.gz		解压xxx.tar.gz	
tar -jxf xxx.tar.bz2		解压xxx.tar.bz2

三、使用Vim文本编辑器
1，三种模式
		          shell

		        命令模式

	       输入模式		末行模式
从命令模式到输入模式：
i	从当前光标前插入一个空字符
o 	在当前光标下新加一空行
从命令模式到末行模式：
:
?
/
从输入模式和末行模式返回命令模式：
esc键
2，vim的打开与退出
（1），vim file 	若file存在则打开file
                   	若file不存在则新建file
（2），翻页（命令模式、插入模式）
PgUp    向上翻动一整页内容 = Ctrl + B
PgDn    向下翻动一整页内容 = Ctrl + F
（3），行内的跳转（命令模式、插入模式）
^	将光标跳转到本行的行首字符  = Home键和数字0
$	将光标跳转到本行的行尾字符  = End键
（4），文件行内的跳转
1G=gg	       跳转到文件的首行
G	       跳转到文件的尾行
#G（#gg）	跳转到文件的#行（命令模式）
:#	      跳转到文件的#行（末行模式）
（5），复制粘贴
yy	复制当前光标所在行
#yy	复制当前光标向下#行
p	粘贴
在末行模式中复制	:1,3y	复制1-3行
(6)，删除操作（在命令模式中）
x	删除光标处的单个字符 = Del
dd	删除光标所在行
#dd	删除#行
d^    从光标处之前删除至行首
d$    从光标处删除到行尾
(7)，字符串的查找（末行模式）
/world	向下查找world
？world   向上查找world
n	定位下一个匹配的字串
N	定位上一个匹配的字串
(8)，撤销编辑
u	 取消最近一次的操作，能多次使用
U	 取消当前行所有的操作
ctrl +r	 对使用u命令撤销操作进行恢复
J	 合并当前行和下一行
(9)，存盘及退出（在末行模式中按）
:q	退出
:w	保存
:wq	保存退出     = ZZ = :x
:X	加密
:wq!	强制保存退出
:! 命令	在vim中执行外面命令
:w file	另存为file
:r file	读入file
:e file        打开其他文件编辑
（10），字符串的替换（末行模式）
:s/old/new         将当前行中查找到的第一个字符“old” 串替换为“new”
:s/old/new/g       将当前行中查找到的所有字符串“old” 替换为“new”
:#,#s/old/new/g    在行号“#,#”范围内替换所有的字符串“old”为“new”
:%s/old/new/gc 	   在整个文件范围内替换所有的字符串“old”为“new并对每个替换动作提醒
（11），末行模式基本操作
:set nu|nonu                   显示/不显示行号
:syntax on|off                 启用/关闭语法高亮
:set hlsearch|nohlsearch       开启/关闭查询结果高亮显示
:set autoindent|noautoindent   启用/关闭自动缩进
在vim ~/.vimrc文件中

########################测试题############################
一、文件内容操作
1，使用cat命令查看/etc/resolv.conf
[root@localhost tmp]# cat /etc/resolv.conf

2，使用more命令查看/etc/mail/sendmail.cf
[root@localhost tmp]# more /etc/resolv.conf

3，使用less命令查看/etc/mail/sendmail.cf
[root@localhost tmp]# less /etc/mail/sendmail.cf

4，对比cat more less区别和特点

5，查看/etc/passwd前5行
[root@localhost tmp]# head -n 5 /etc/passwd

6，查看/etc/passwd尾5行
[root@localhost tmp]# tail -n 5 /etc/passwd

7，查看/etc/passwd的第8-12行
[root@localhost tmp]# head -n 12 /etc/passwd | tail -n 5

8，统计系统中有多少个账户
[root@localhost tmp]# wc -l  /etc/passwd 

9，计算/etc目录下.conf配置文件的个数
[root@localhost tmp]# find /etc -name *.conf | wc -l

10，显示/etc/hosts中127.0.0.1的内容
[root@localhost tmp]# grep 127.0.0.1 /etc/hosts

11，显示/etc/passwd中以root开头的内容
[root@localhost tmp]# grep ^root /etc/passwd

12，显示/etc/passwd中以bash结尾的内容
[root@localhost tmp]# grep bash$ /etc/passwd

13，去除/etc/hosts.allow中的空行，把结果显示出来
[root@localhost tmp]# grep -v "^$" /etc/hosts.allow

14，查找Linux识别的eth接口的信息
[root@localhost ~]# dmesg | grep eth --color

15，显示/etc/hosts里面不以#号开头的内容
[root@localhost tmp]# grep -v ^# /etc/hosts.allow

16，计算以/bin/bash作登陆shell的用户个数
[root@localhost tmp]# grep /bin/bash$ /etc/passwd | wc -l

17，查找/etc/hosts中包含127.0.0.1或者localhost的内容
[root@localhost tmp]# grep -E "127.0.0.1|localhost6" /etc/hosts

二、压缩和归档
1，以易读的属性并长格式显示/root下的内容将结果重定向到/root/gztest.txt里面
ls -l /root> /root/gzest.txt
2，分别使用gzip和bzip2和zip对/root/gztest.txt进行压缩和解压
[root@localhost ~]# gzip gztest.txt 
[root@localhost ~]# gzip -d gztest.txt.gz 
[root@localhost ~]# bzip2 gztest.txt 
[root@localhost ~]# bzip2 -d gztest.txt.bz2 
[root@localhost ~]# zip gztest.txt.zip gztest.txt 
[root@localhost ~]# unzip gztest.txt.zip

3，把/etc/mail打包并压缩到/root/mail.tar.gz
[root@localhost ~]# tar -zcf /etc/mail.tar.gz /etc/mail

4，把/etc/mail打包并压缩到/root/mail.tar.bz2
[root@localhost ~]# tar -jcf /etc/mail.tar.bz2 /etc/mail

5，将mail.tar.gz解压到/tmp下，递归查看/tmp/etc下的内容，然后删除/tmp/etc目录
[root@localhost ~]# tar -zxf /etc/mail.tar.gz   -C /tmp/
[root@localhost ~]# ls -R /tmp/etc


6，将mail.tar.bz2解压到/tmp下，递归查看/tmp/etc下的内容，然后删除/tmp/etc目录
[root@localhost ~]# tar -jxf /etc/mail.tar.bz2   -C /tmp/

7，分别查看mail.tar.gz与mail.tar.bz2文件里面内容
[root@localhost ~]# tar -ztf /etc/mail.tar.gz
[root@localhost ~]# tar -jtf /etc/mail.tar.bz2

三、使用vim
1，请在 /tmp 这个目录下建立一个名为 vimtest 的目录
[root@localhost ~]# mkdir /tmp/vimtest

2，进入vimtest 这个目录当中
[root@localhost ~]# cd /tmp/vimtest/

3，将 /etc/man.config 复制到本目录底下

4，使用 vim 打开本目录下的 man.config
[root@localhost vimtest]# cp /etc/man.config .

5，在 vim 中设定一下行号
末行模式：set nu
6，移动到第 58 行，向右移动 40 个字元，请问你看到的双引号内是什么目录？
58 gg                         40—>
7，移动到第一行，并且向下搜寻一下‘ bzip2 ’这个字串，请问他在第几行？
   /bzip2   
8，我要将 50 到 100 行之间的‘小写 man 字串’改为‘大写 MAN 字串’，并且一个一个挑选是否需要修改，如何下达指令？如果在挑选过程中一直按‘y’， 结果会在最后一行出现改变了几个 man 呢？
50,100s/nam/MAN/gc
9，修改完之后，突然反悔了，要全部复原，有哪些方法？
u
10，我要复制 65 到 73 这九行的内容(含有MANPATH_MAP)，并且贴到最后一行之后
65,73y              定位到65 9yy               G 
11，21 到 42 行之间的开头为 # 符号的注解资料我不要了，要如何删除？
21,42d
12，将这个档案另存成一个 man.test.config 的档名
mv /etc/man.conf /etc/man.test.config
13，去到第 27 行，并且删除 15 个字元，结果出现的第一个单字是什么？
27gg   15dd
14，在第一行新增一行，该行内容输入‘I am a student...’
15，储存后离开吧






