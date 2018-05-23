一、命令行基础
1，Linux命令格式
命令字  [选项]  [参数]
[] 表示里面内容可有可无
选项：如果是单个字符，用-	      # ls -l
      如果是一个单词，用--	      # ls --color
      多个单个字符的选项可以合并一个-  # ls -l -h = # ls -lh
2，tab键
	命令补全
	路径补全
	判断命令和路径是否有错
3，快捷键
Ctrl + u：清空至行首
Ctrl + k：清空至行尾
Ctrl + l：清空整个屏幕 = #clear
Ctrl + c：废弃当前编辑的命令行
4，获取帮助命令
type：查看内部命令和外部命令
help 内部命令
外部命令 --help
man 命令 
# makewhatis	     生成数据库
# man -f = # whatis  查看具体后面接的这个命令支持哪些格式的帮助
数字可以帮助我们了解或者是直接查询相关的资料
1	指令或可执行文件
5	文件格式
8	系统管理员可用的指令

二、目录和文件基本管理
1，基本命令
# pwd	显示我现在哪里（目录）
路径
	绝对路径：以/开始的路径
	相对路径：不以/开始的路径
特殊目录：
	.	当前目录
	..	上一层目录
	../..	上一层目录的上一层
	-	前一个工作目录
	~	"目前用户身份”所在主目录
	~账户名	这个账户的主目录
cd：切换目录
	# cd ~root = # cd ~ = # cd
	# cd .
	# cd ..
	# cd ../..
	# cd -
ls：查看文件与目录
	-a      查看所有文件(包含隐藏文件)
	-l = ll 长格式显示
	-ld	长格式显示目录
	-lh	以字节单位（K、M等）显示信息
	-R	递归显示内容
通配符：
	*：匹配任意多个字符
	?：匹配单个字符
	[n-m]：匹配连续多个字符中的一个
	{a,x,y}：匹配不连续的多个字符中的
# alias					查看系统别名
# alias byebye="shutdown -h now"   	自定义别名

2,查看文件目录大小
du -sh 

3，创建目录和文件
mkdir：新建目录
	-p	递归创建
rmdir：删除空目录
	-p	递归删除
touch：创建文件，更新时间戳
ln  -s 	创建软链接
4，复制/删除/移动操作 
cp：复制文件或目录
	-a = -pdr	
	-f	强制
	-p	属性一同复制过去
	-r	复制目录
		
rm：移除文件或目录
	-f	强制
	-r	递归删除

mv：移动文件与目录，或更名
	
三、程序和文件检索
1，执行命令路径的变量PATH
echo $PATH	查看PATH的值
作用：
	执行任何命令的时候会去从PATH的值（路径）中去搜寻是否有该命令。有就执行，没有就告诉你找不到这个命令

2，命令与文件查询
which		专门用于查找命令
whereis		既能查找命令，也能查找文件
locate		查找速度快，需要通过updatedb生成数据库		


find:
格式：find [路径] [条件]
默认不指定路径，就是当前路径
	-type 	类型
		f	文件
		d	目录
		l	链接

	-name	名字
		-a	多个条件同时满足
		-o	多个条件满足一条即可


	-size 	大小（单位c表示byte，k表示1024bytes）
		+ 	大于
		-	小于


	-mtime 文件内容修改
	  n   n为数字，意义在n天之前的“一天之内”被更改过的文件
	  +n  列出在n天之前（不含n天本身）被更改过的文件名
	  -n  列出在n天之内（含n天本身）被更改过的文件名
		              4
	                    <--->		                                                       -4|---------------------->
<--------------------------|+4		<--------------|-----|-----|-----|-----|-----|-----|-----|
	       7     6	   5     4     3     2     1    现在
		eg:	
	+4代表大于等于5天前的文件名   find /var -mtime +4
	-4代表小于等于4天内的文件名    find /var -mtime -4
	4则是代表4~5那一天的文件名    find /var -mtime 4
-exec command	
eg： find /boot -size +2048k -exec ls -l {} \;
{}		find找到的内容
-exec	到\;	代表find额外命令开始到结束
;		特殊字符，需要转义



########################测试题############################
一、命令行基础
1，查看cd和mkdir是属于内部命令还是外部命令
[root@localhost ~]# type cd
cd is a shell builtin
[root@localhost ~]# type mkdir
mkdir is hashed (/bin/mkdir)

2，使用help查看cd的帮助信息
[root@localhost ~]# help cd

3，使用help查看mkdir的帮助信息
[root@localhost ~]# mkdir --help

4，使用man查看passwd命令的帮助信息
[root@localhost ~]# whatis passwd
passwd               (1)  - update user's authentication tokens
passwd               (5)  - password file
passwd              (rpm) - The passwd utility for setting/changing passwords using PAM
passwd [sslpasswd]   (1ssl)  - compute password hashes
[root@localhost ~]# man 1 passwd

5，使用man查看passwd文件的帮助信息
[root@localhost ~]# man 5 passwd

二、目录和文件基本管理
1，显示现在什么位置
[root@localhost ~]# pwd

2，进入/etc/sysconfig/network-scripts下
[root@localhost ~]# cd /etc/sysconfig/network-scripts

3，长格式并提供易读的属性显示/boot下的vmlinuz开头的文件
[root@localhost network-scripts]# ls -lh /boot/vmlinuz*

4，列出/etc目录属性
[root@localhost network-scripts]# ls -ld /etc

5，递归显示/boot目录下的文件和内容
[root@localhost network-scripts]# ls -R /boot

6，显示root下面所有文件包括隐藏文件
[root@localhost network-scripts]# ls -a /boot

7，进入/tmp目录，删除所有文件和目录，创建file1.txt file2.txt file3.txt file13.txt filea.txt fileab.txt
[root@localhost ~]# cd /tmp
[root@localhost ~]# rm -rf *
[root@localhost ~]# rm -f *
[root@localhost tmp]# touch file1.txt

8，显示file开头的，以.txt结尾的，中间2个字符的文件
[root@localhost tmp]# ls -l file??.txt

9，显示file开头的，以.txt结尾的，中间是单个数字的文件
[root@localhost tmp]# ls -l file[0-9].txt

10，显示file开头的，以.txt结尾的，中间部分可能是1 3 a ab的文件
[root@localhost tmp]# ls -l file{1,3,a,ab}.txt

11，定义alias别名，设置myls=ls -lhA
[root@localhost tmp]# alias myls="ls -lhA"

12，查看/boot和/etc/pki分别占用多大空间
[root@localhost ~]# du -sh /boot /etc/pki

13，创建/vod/movie/cartoon，递归显示/vod目录结构
[root@localhost ~]# mkdir -p /vod/movie/cartoon
[root@localhost ~]# ls -R /vod

14，把system-config-network-tui链接成/sbin/netconfig
 ln -s /usr/sbin/system-config-network-tui /sbin/netconfig

15，把/boot/grub  /etc/host.conf拷贝到/root/Desktop下，在尾部添加标记的方式列出Desktop目录下的内容（ls -F）
[root@localhost etc]# cp -r /boot/grub  /etc/host.conf /root/Desktop/
[root@localhost etc]# ls -F /root/Desktop/

16，删除Desktop下的grub和host.conf
[root@localhost etc]# rm -rf /root/Desktop/grub  /root/Desktop/host.conf 

17，创建/root/ls-man.txt文件，在将这个文件移动到桌面
[root@localhost tmp]# touch /root/ls-man.txt
[root@localhost tmp]# mv /root/ls-man.txt /root/Desktop/

18，把ls-man.txt改名为manls.txt
[root@localhost tmp]# mv /root/Desktop/ls-man.txt /root/Desktop/manls.txt

三、程序和文件检索
1，PATH作用，查看PATH的值
[root@localhost tmp]# echo $PATH 

2，查找shutdown这个命令的绝对路径
[root@localhost tmp]# which shutdown

3，通过whereis搜索 rm
[root@localhost tmp]# whereis rm

4，创建myhttpd.conf文件，使用locate查找，是否能够查找到这个文件？
[root@localhost tmp]# touch myhttpd.conf
[root@localhost tmp]# locate myhttpd.conf

5，更新数据库 /var/lib/mlocate/mlocate.db，使用locate查找，是否能够查找到这个文件？
[root@localhost ~]# updatedb
[root@localhost ~]# locate myhttpd.conf

6，删除myhttpd.conf文件，使用locate查找，是否能够查找到这个文件？
[root@localhost ~]# rm myhttpd.conf
rm：是否删除 一般空文件 “myhttpd.conf”? y
[root@localhost ~]# locate myhttpd.conf

7，查找/boot下的链接文件
[root@localhost ~]# find /boot -type l

8，查找/boot下的目录
[root@localhost ~]# find /boot -type d

9，查找/etc下名字叫resol开头的，以.conf结尾的文件
[root@localhost ~]# find /etc -type f -a -name resol*.conf

10，查找/dev下的字符设备文件，并且名字叫tty1 tty2 tty3的
[root@localhost ~]# find /dev -type c -a -name tty[1,2,3]

11，以易读的属性并长格式显示/boot下以.img结尾的文件
[root@localhost ~]# find /boot -type f -a -name *.img -exec ls -lh {} \;
[root@localhost ~]# ls -lh /boot/*.img

12，查找/boot下以.img结尾的并且大于2M的文件
[root@localhost ~]# find /boot -type f -a -name *.img -size +2M

13，查找系统上面24小时内变动过的文件
[root@localhost ~]# find / -mtime 1

14，查找/var下大于等于5天前变动过的文件名
[root@localhost ~]# find /var -mtime +4

15，查找/var下小于等于4天内变动过的文件名
[root@localhost ~]# find /var -mtime -4

16，查找/var下第4-5天那一天变动过的文件名
[root@localhost ~]# find /var -mtime 4

17，查找/boot下大于3M的文件并把它长格式显示出来
[root@localhost ~]# find /boot -type f -size +2M -exec ls -lh {} \;

