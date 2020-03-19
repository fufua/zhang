# day27磁盘管理下部

![](https://cdn.nlark.com/yuque/0/2019/png/194754/1546516031364-8facb71c-4091-43f1-ab1c-8014e1afceea.png#align=left&display=inline&height=298&originHeight=761&originWidth=2113&status=done&width=827)

[day27磁盘管理下部.xmind](https://www.yuque.com/attachments/yuque/0/2019/xmind/194754/1554020415432-d2322ff4-99e7-45aa-a0bb-2a5562aa98e9.xmind?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2019%2Fxmind%2F194754%2F1554020415432-d2322ff4-99e7-45aa-a0bb-2a5562aa98e9.xmind%22%2C%22name%22%3A%22day27%E7%A3%81%E7%9B%98%E7%AE%A1%E7%90%86%E4%B8%8B%E9%83%A8.xmind%22%2C%22size%22%3A141682%2C%22type%22%3A%22application%2Fvnd.xmind.workbook%22%2C%22progress%22%3A%7B%22percent%22%3A0%7D%2C%22status%22%3A%22done%22%2C%22percent%22%3A0%2C%22id%22%3A%221bs5D%22%2C%22card%22%3A%22file%22%7D)

![](https://cdn.nlark.com/yuque/0/2019/png/194754/1546516138799-98763c99-e90b-4784-9a73-97a9af2c53c3.png#align=left&display=inline&height=892&originHeight=1034&originWidth=959&status=done&width=827)

![](https://cdn.nlark.com/yuque/0/2019/png/194754/1546516088591-caac8ddd-d134-4188-9387-3b0367cf22fa.png#align=left&display=inline&height=247&originHeight=2276&originWidth=7632&status=done&width=827)

1 磁盘组成
2 RAID 级别
3 如何让系统更安全
```bash
# 和md5类似 ，加密算法更复杂
[root@oldboy01 yum.repos.d]# sha
sha1sum    sha224sum  sha256sum  sha384sum  sha512sum  


http://lidao.blog.51cto.com/3388056/1910889
如何防止系统中木马
```

4 磁盘接口
5 如何进行计算

今天：
1 如何查看内存使用情况
2 磁盘分区
3 进行分区 格式化  挂载
4 故障案例
5 一大波磁盘相关命令

# 查看内存使用情况
```bash
[root@oldboy01 yum.repos.d]# free -h
            total       used       free     shared    buffers     cached
Mem:          474M       389M        84M       232K        15M       286M
-/+ buffers/cache:        87M       387M
Swap:         767M       920K       767M
剩余内存 387M = free 84M + 15M buffers +286M cached
```
linux把你使用过的命令或文件 替你缓存（buffer cache）起来，提高下次使用速度
写buffer
读cache

# 磁盘分区fdisk
linux启动流程：

      1. 开机自检BIOS
      1. MBR引导
      1. GRUB菜单
      1. Kernel加载内核
      1. 运行init 进程    第一个进程
      1. /etc/inittab    读取运行级别
      1. /etc/rc.sysinit  系统初始化
      1. /etc/rc.d/rc3.d   根据运行级别 启动对应服务（开机自启动）
      1. mingetty 		登录界面



磁盘的引导扇区   0磁头 0磁道 1扇区
MBR引导     0头0道1扇区前446字节
MBR（Mster Boot Record）   主引导记录  引导系统启动
DPT（Disk Partition Table）   磁盘分区表 记录着磁盘分区从哪里开始到哪里结束

主分区(primary) 每个分区占用16个字节的分区表
扩展分区(extended) 无法直接使用。 再创建逻辑分区
逻辑分区(logical)   存放数据

磁盘分区的命名规则

第1块sas硬盘的第一个主分区    /dev/sda1
第2块sata硬盘的第2个主分区   /dev/sdb2
第3块sata硬盘的第1个逻辑分区 /dev/sdc5

```bash
#虚拟机添加了两个硬盘
[root@oldboy01 ~]# fdisk -l|grep sd[a-c]
Disk /dev/sda: 21.5 GB, 21474836480 bytes
/dev/sda1   *           1          26      204800   83  Linux
/dev/sda2              26         124      786432   82  Linux swap / Solaris
/dev/sda3             124        2611    19979264   83  Linux
Disk /dev/sdb: 213 MB, 213909504 bytes
Disk /dev/sdc: 213 MB, 213909504 bytes
```
```bash
fdisk  -c -u
      -u     When listing partition tables, give sizes in sectors  instead  of
             cylinders.
				磁盘分区的时候 以扇区为单位 默认是按照 柱面
      -c     Switch off DOS-compatible mode. (Recommended)
				关闭dos兼容模式
```

fdisk –cu

```bash
[root@oldboy01 ~]# fdisk -cu /dev/sdb
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0x914f7c27.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.
Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)
Command (m for help):
#fdisk 内部命令
m 显示帮助
n new 创建分区
p 显示所有分区信息
d 删除分区
w 保存并退出
q 不保存退出
Command (m for help): n      
Command action
  e   extended					# 扩展分区
  p   primary partition (1-4)  #主分区
Invalid partition number for type `l'    # 类型分区号无效
p
Partition number (1-4):   #分区号码
Partition number (1-4): 1
First sector (2048-417791, default 2048):  # 从哪里开始 （回车使用默认）
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-417791, default 417791): +10M   #结束位置
Command (m for help): p
Disk /dev/sdb: 213 MB, 213909504 bytes
64 heads, 32 sectors/track, 204 cylinders, total 417792 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x914f7c27
  Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048       22527       10240   83  Linux
。。。。。。。。
Command (m for help): p
Disk /dev/sdb: 213 MB, 213909504 bytes
64 heads, 32 sectors/track, 204 cylinders, total 417792 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x914f7c27
  Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048       22527       10240   83  Linux
/dev/sdb4           22528      417791      197632    5  Extended
/dev/sdb5           24576      126975       51200   83  Linux
/dev/sdb6          129024      231423       51200   83  Linux
```

增加硬盘200MB  硬盘创建一个分区挂载到/data 目录

1. 创建分区   （上面已经创建好）


  2 通知系统sdb 磁盘分区表变化  partprobe /dev/sdb
  3 创建文件系统 （格式化）
		make filesystem
		mkfs
```bash
对每个房间装修（磁盘分区）
[root@oldboy01 data]# mkfs.ext4 /dev/sdb1

This filesystem will be automatically checked every 21 mounts or
这个磁盘分区会被自动检查   每挂载21次或180天 会进程一次磁盘检查
180 days, whichever comes first.  Use tune2fs -c or -i to override.
自己创建的磁盘分区关闭磁盘检查。
[root@oldboy01 data]# tune2fs -c 0 -i 0 /dev/sdb1  # 关闭磁盘自动检查
tune2fs 1.41.12 (17-May-2010)
Setting maximal mount count to -1
Setting interval between checks to 0 seconds

-c 每挂载多少次 进行一次磁盘检查     0 是关闭
-i 每过多少天 进行一次磁盘检查    0 是关闭
```
4 挂载
```bash
[root@oldboy01 data]# mount /dev/sdb1 /data1
# 检查
[root@oldboy01 data]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        19G  7.1G   11G  40% /
tmpfs           238M     0  238M   0% /dev/shm
/dev/sda1       190M   40M  141M  22% /boot
/dev/sdb1       194M  1.8M  182M   1% /data1

报错：
[root@oldboy01 data]# mount /dev/sdb1 /data1
mount: you must specify the filesystem type #挂载必须指定文件系统类型

```
6 永久挂载
方法一 /etc/fstab   开机自动挂载
```bash
UUID=0c9291ad-1bc9-48a1-85b3-bce1314499ac  /           ext4          defaults        1           1
UUID=251cad76-385f-489d-b315-de8b588fdc57  /boot       ext4          defaults        1           2
UUID=c4bdfe6a-678b-4ea6-8bbc-53e7ae882417  swap        swap          defaults        0           0
tmpfs                                      /dev/shm    tmpfs         defaults        0           0
devpts                                     /dev/pts    devpts        gid=5,mode=620  0           0
sysfs                                      /sys        sysfs         defaults        0           0
proc                                       /proc       proc          defaults        0           0
设备名称（分区）			 挂载点(目录) 文件系统类型   挂载参数   是否进行备份 是否开机磁盘检查
/dev/cdrom
/dev/sdb1
dev/sdb1   			/data1		 ext4 		defaults		  0 	0

# 查看uuid 对应设备
[root@oldboy01 data]# blkid
/dev/sda3: UUID="0c9291ad-1bc9-48a1-85b3-bce1314499ac" TYPE="ext4"
/dev/sda1: UUID="251cad76-385f-489d-b315-de8b588fdc57" TYPE="ext4"
/dev/sda2: UUID="c4bdfe6a-678b-4ea6-8bbc-53e7ae882417" TYPE="swap"
/dev/sdb1: UUID="12d89b97-accd-459b-88fe-bed0a40d6a81" TYPE="ext4"
```
方法二 /etc/rc.local 增加  bin/mount /dev/sdb1 /data1

磁盘使用小结：
1分区
2 格式化
3 挂载

定时任务重启tomcat  发现重启失败 报错， 定时任务脚本中加入了环境变量

# 故障案例： java 程序占用大量内存，开始使用swap， swap不足
## 增加swap
```
创建一个文件成为swap
[root@oldboy01 data]# free -h
            total       used       free     shared    buffers     cached
Mem:          474M       140M       334M       236K        23M        41M
-/+ buffers/cache:        74M       399M
Swap:         767M         0B       767M
1 创建一个100M 的文件
[root@oldboy01 data]# dd if=/dev/zero of=/tmp/100m bs=1M count=100
if== input file 从哪里获取    of  output  file 输出到哪里  block size每次复制多少  count 复制多少次
/dev/zero  不断输出 零
/dev/null  黑洞
[root@oldboy01 data]# file /tmp/100m  # data类型
/tmp/100m: data

2 创建swap 让这个文件成为swap（格式化）
[root@oldboy01 data]# mkswap /tmp/100m  #创建swap类型
mkswap: /tmp/100m: warning: don't erase bootbits sectors
       on whole disk. Use -f to force.
Setting up swapspace version 1, size = 102396 KiB
no label, UUID=fef00b6f-81ca-403b-8b95-78bfcbeca312
[root@oldboy01 data]# file /tmp/100m  # 显示文件类型  是swap类型
/tmp/100m: Linux/i386 swap file (new style) 1 (4K pages) size 25599 pages

3 激活swap 分区
[root@oldboy01 data]# swapon /tmp/100m
[root@oldboy01 data]# free -h
            total       used       free     shared    buffers     cached
Mem:          474M       297M       177M       236K        24M       193M
-/+ buffers/cache:        79M       395M
Swap:         867M         0B       867M
[root@oldboy01 data]# swapon –s    # 显示swap 组成情况（磁盘分区 和 文件）
Filename				Type		Size	Used	Priority
/dev/sda2                               partition	786428	0	-1
/tmp/100m                               file		102396	0	-2

4 永久增加
/etc/rc.local   增加 sbin/swapon /tmp/100m
/etc/fstab      类型和挂载点都是 swap
```

增加swap小结
1 创建一个文件
2 文件---》 swap
3 激活 和 永久生效

fdisk  支持2TB以内的硬盘     只支持MBR磁盘分区表
# parted 支持2TB以上的硬盘	MBR（主分区最多4个）,GPT（无限 接近100）

```bash
[root@oldboy01 data]# parted /dev/sdc
print    显示分区信息
报错
(parted) p                                                                
Error: /dev/sdc: unrecognised disk label   没有分区表
mktable  mklabel创建磁盘分区表  gpt msdos（mbr）
mkpart   创建分区
rm   	 删除分区
q  		 退出不保存

(parted) mktable gpt   # 创建gpt分区表
# 提示已经存在  磁盘分区表  在/dev/sdc  是不是继续                                                
Warning: The existing disk label on /dev/sdc will be destroyed and all data on this disk
will be lost. Do you want to continue?
Yes/No? y  
(parted) mkpart primary 0 10     #创建分区 默认是M 为单位                                        
Warning: The resulting partition is not properly aligned for best performance.
Ignore/Cancel? i    
(parted) p    #查看                                                     
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 214MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name     Flags
1      17.4kB  10.0MB  9983kB               primary
(parted) q   #退出      
#格式化已经分好的区  mkfs.ext4 /dev/sdc1
# 挂载 mount /dev/sdc1 /sdc1
# 开机自动挂载 echo ‘/dev/sdc1 /sdc1 ext4 defaults 0 0’ >> /etc/fstab
                                                    
Information: You may need to update /etc/fstab.  非交互式创建分区
[root@oldboy01 data]# parted /dev/sdc  mkpart primary 0 10 ignore
Warning: The resulting partition is not properly aligned for best performance.
Information: You may need to update /etc/fstab.        
[root@oldboy01 data]# parted /dev/sdc mkpart primary 10 20 ignore
Warning: You requested a partition from 10000kB to 20.0MB.                
The closest location we can manage is 10.0MB to 10.5MB.
Is this still acceptable to you?
parted: invalid token: ignore
Yes/No? y                                                                 
Warning: The resulting partition is not properly aligned for best performance.
Ignore/Cancel? i                                                         
Information: You may need to update /etc/fstab.                          
[root@oldboy01 data]# parted /dev/sdc p   # 显示分区信息
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 214MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number  Start   End     Size    File system  Name     Flags
1      17.4kB  10.0MB  9983kB               primary
3      10.0MB  10.5MB  485kB                primary
2      10.5MB  19.9MB  9437kB               primary


```

总结：
0 磁盘分区表   主分区 扩展分区 逻辑分区
1 磁盘分区命名规则
2 磁盘分区过程
3 分区格式化挂载  
增加swap
4 如何查看内存使用情况


预习 ：
常见的文件系统
磁盘相关命令
shell编程 – 判断 循环
sed
awk   awk数组
网络



