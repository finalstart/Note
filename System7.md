一、磁盘分区及格式化
1、fdisk -l	查看当前磁盘分区
2、fdisk /dev/xdy	对/dev/xdy进行分区
	n 创建新分区
	p 查看分区
	d 删除分区
	w 保存
	q 退出
	t 改变文件类型
		82 swap
		83 ext3	（默认）
		8e lvm
		fd raid
		b  fat32
3、partprobe /dev/xdy 		获取新分区表
4、mkfs -t 文件类型 /dev/xdyz	格式化  =  mkfs.文件类型 /dev/xdyz
5、mkswap /dev/xdyz		把/dev/xdyz格式化成swap类型分区
	-L 设置label
[root@localhost ~]# mkswap -L SWAP-sdb5 /dev/sdb5
Setting up swapspace version 1, size = 2006929 kB
LABEL=SWAP-sdb5, no uuid

   swapon /dev/xdyz		启用/dev/xdyz到swap分区中
   swapoff /dev/xdyz		停用/dev/xdyz
二、文件系统管理
1，e2label /dev/sdb1 "disk2part1"	设置卷标
   e2label /dev/sdb1			查看卷标
   e2label /dev/sdb1 ""			删除卷标
2，blkid		查看块设备属性（UUID）
3，mount
	-a  把fstab里面已配置的挂载 
	-t  指定挂载类型
	-o  指定挂载参数
  	-L  使用卷标名挂载
	-o loop	挂载光盘镜像文件
[root@localhost Desktop]# mount -o loop /root/Desktop/linux.iso /mnt/
[root@localhost Desktop]# mount | grep mnt
/root/Desktop/linux.iso on /mnt type iso9660 (rw,loop=/dev/loop0)

	--bind	目录挂载目录下
[root@localhost ~]# mkdir -p /media/tools/src
[root@localhost ~]# mount --bind /media/tools/src/ /usr/src/
[root@localhost ~]# mount | grep src
/media/tools/src on /usr/src type none (rw,bind)

4，df
	-h	易读方式查看分区大小
	-T	显示时显示文件类型
5，umount	卸载
6，/etc/fstab	开机自动挂载文件
格式：
设备文件（卷标名/UUID） 挂载点 类型 挂载参数 备份标记 检测顺序
/dev/sdb5		        swap  swap defaults        0        0
7，autofs触发挂载
/etc/auto.master	autofs主配置文件
格式：
挂载点父目录	挂载配置文件（自定义）
	
/etc/auto.xxx	挂载配置文件   
格式：
挂载点子目录(不需要创建)	挂载参数		:设备名

# /etc/init.d/autofs stop	停止autofs服务
# /etc/init.d/autofs start	开启autofs服务
	

一、磁盘分区及格式化
1，查看当前磁盘分区情况（使用2种方法）
[root@localhost ~]# df -h
[root@localhost ~]# mount

2，添加一块80GB的SCSI磁盘，划分2个20GB的主分区，剩余作为扩展分区使用
3，新建2个逻辑分区，分别为2GB和10GB
Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1        2433    19543041   83  Linux
/dev/sdb2            2434        4866    19543072+  83  Linux
/dev/sdb3            4867       10443    44797252+   5  Extended
/dev/sdb5            4867        5110     1959898+  83  Linux
/dev/sdb6            5111        6327     9775521   83  Linux

4，将第一个逻辑分区标记为SWAP，并格式化为SWAP分区，卷标为disk2part5
Command (m for help): t
Partition number (1-6): 5
Hex code (type L to list codes): 82
Changed system type of partition 5 to 82 (Linux swap / Solaris)
[root@localhost ~]# mkswap -L disk2part5 /dev/sdb5

5，前二个主分区格式化成EXT3，第二个逻辑分区格式化成支持Windows的分区
Command (m for help): t
Partition number (1-6): 6
Hex code (type L to list codes): b
Changed system type of partition 6 to b (W95 FAT32)
[root@localhost ~]# mkfs.ext3 /dev/sdb1
[root@localhost ~]# mkfs.ext3 /dev/sdb2
[root@localhost ~]# mkfs.vfat /dev/sdb6

6，把格式化的交换分区扩展到当前系统当中
[root@localhost ~]# swapon LABEL=disk2part5

二、文件系统管理
1，设置卷标（需要先格式化）
	第一个主分区disk2part1
   	第二个主分区disk2part2
[root@localhost ~]# e2label /dev/sdb1 disk2part1
[root@localhost ~]# e2label /dev/sdb2 disk2part2

2，查看/dev/sdb下分区的UUID
[root@localhost ~]# blkid /dev/sdb*

3，创建/media/{tools,game,windows}目录，把第一个主分区挂载到这个目录
[root@localhost ~]# mount /dev/sdb1 /media/game

4，把RHEL5.9系统盘里面的images/boot.iso挂载到/media/isofs
-o loop	挂载光盘镜像文件
5，把/usr/src目录挂载到/media/tools/src下
[root@localhost ~]# mount --bind   /usr/src   /media/tools/src

6，查看当前系统磁盘使用情况
[root@localhost ~]# df -h
7，卸载刚才所有挂载
[root@localhost ~]# umount -a

8，通过/etc/fstab把刚才的分区挂载
	/dev/sdb1  /media/tools
	/dev/sdb2  /media/game
	/dev/sdb5  swap
	/dev/sdb6  /media/windows
[root@localhost ~]# vim /etc/fstab
UUID="68bfe9d2-b312-41c6-84ae-e23b3fb17c1c"  /media/tools ext3 defaults   0 0
UUID="ae6b1cfd-f9cb-4c10-b4bd-121517740b8e" /media/game ext3 defaults  0 0
UUID="52A6-DE1A"    /media/windows                                         vfat  defaults  0 0
LABEL="disk2part5"      swap                                                          swap  defaults  0 0
[root@localhost ~]# mount -a

9，配置autofs，让自动挂载/dev/sdb1、/dev/sdb2、/dev/sdb6
[root@localhost ~]# vim /etc/auto.master
/media    auto.media
[root@localhost ~]# vim /etc/auto.media
tools   -fstype=ext3  :/dev/sdb1
game    -fstype=ext3  :/dev/sdb2
windows -fstype=vfat  :/dev/sdb6
[root@localhost ~]# service autofs stop
[root@localhost ~]# service autofs start


