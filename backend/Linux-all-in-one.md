---
layout: post
title: "Linux基础套餐 原理+实操"
permalink: /linux-all-in-one
---



Linux的诞生故事总结起来就是：**反收费，促开源**。Linux最初只是局限在硬件接口层，当Linus选择把Linux加入GNU（Mr. Stallman发起的一个开源计划）之后，就有源源不断的开发者为其提供了各种外围的软件。就是说，对于一个操作系统来说，最重要的是如何去高效调配硬件资源（内存、CPU），而界面、用户命令什么的都是次要的。

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220514215712312.png" alt="image-20220514215712312" style="zoom: 33%;" />



### 文件系统

---

Linux中管它软件还硬件一切皆文件

![image-20220515133734602](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220515133734602.png)

#### 分区挂载

Linux中支持为一块分区指定一个存放文件目录（文件夹），这个过程叫做挂载。一块硬盘可以在逻辑上划分为多个分区，也可以单独作为一个分区挂载在一个文件目录之下。

整盘分区挂载在/home下，分盘分区挂在/boot目录下

#### 各目录的含义

**/bin**：各种命令的源文件所在地，诸如cd、ls等等。是/usr/bin目录的快捷方式

/sbin：超级用户的命令。是/usr/sbin目录的快捷方式

/lib：库目录，系统和程序需要的共享库文件 。是/usr/lib目录的快捷方式

 **/usr**：用户文件目录

/boot：用于挂载分区

/dev：device的简称，硬件设备管理目录，诸如CPU

**/etc**：系统管理需要的配置文件及一些软件的配置文件，诸如MySQL

/home：用户的主目录

/root：超级管理员用户目录

**/opt**：optional可选目录，给第三方软件包的安装位置

/media：可移动媒体目录，U盘和光驱

/mnt：外部挂载目录，移动化存储设备的另外一个挂载点，如硬盘

/proc：进程目录，存放当前线程的相关文件

/run：存放当前系统运行以来的所有信息

/srv：存放和系统服务相关的信息

/sys：系统硬件相关的信息

**/tmp**：临时目录，临时操作的东西放在这里

**/var**：可变目录，存放经常会修改的东西，诸如日志log

#### 文件目录类操作命令

**文件夹操作**

pwd		打印当前目录的绝对路径

cd /		切换到根目录

cd ..		切换到上一级目录

cd ../name/		切换到与当前目录同级的其他目录

ls -a		显示全部文件包括以.开头的隐藏文件

ls -l = ll		显示全部文件及其属性

mkdir [相对或绝对路径] [...]		无法直接嵌套创建

mkdir  -p		嵌套创建目录

rmdir -p		嵌套删除

**文件操作**

touch [name]		创建一个文件

cp [options] source dest		

\cp		直接覆盖文件如果存在的话；\是Linux的原生命令

rm [-f 强制删除不提示 -r 递归删除目录下的全部文件]		删除文件

rm -f ./*		删除当前目录下的全部文件

mv [old] [new]		移动或改名

**查看文件**

cat 		单屏查看

more		多屏多页查看	空格或-f-b翻页

less		

echo  “content” >> name.txt

\>>内容追加 	> 内容覆盖

 head/tail -n 5 file_name		默认是前/后10行；-f选项是后台线程挂起持续监控当前文件的变化，这时候如果重写了该文件则会报错

**软链接/符号链接**

ln -s [file_path] [soft link name]  # e.g ln -s /tmp/example/hi.txt ./hi		创建软连接（快捷方式）

**其他**

history		查看历史命令

history -c		清空历史命令



#### 查找

find [搜索范围] [选项]

-size +10M 大于10M的文件

![image-20220516153857184](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220516153857184.png)



locate [file_name]	从Linux所有文件名和路径映射的数据库中查找文件。在使用之前需要先更新一下数据库updatedb



**管道使用**

grep -n [key] [file_name]		从指定文件中查找指定字符

bloc | bloc 前一个的输出结果作为后面的参数，例如：

ls | grep -n *.txt		查找当前目录下所有txt文件

netstat -anp | grep pid		前面的输出是全部的网络使用信息，在这里面找到指定pid的网络占用情况 



#### 压缩解压

**gz解压缩**  

只能压缩文件，不保留源文件

gzip [file_name]

gunzip [file_name]

**zip解压缩**

zip -r [directory_name]		压缩目录

unzip -d [path]		指定解压路径

**tar解压缩**

tar [option] XXX.tar.gz [file or directory] [...]

解压常用 -zxvf -C [directory]

压缩常用 -zcvf

![image-20220516160323179](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220516160323179.png)









### 系统管理

---

#### 服务管理

ls /usr/sbin/ | grep service		从某个目录中查找某个文件，管道的使用

/ect/init.d		查看当前正在运行的服务的守护进程 daemon

service [service_name] start|stop|restart|status		服务管理

systemctl  start|stop|restart|status|enable(开机自启动disable)  [service_name] 		服务管理

systemctl list- unit- files		列出当前全部服务



#### 运行级别

setup		配置服务的开机自启动

init进程是系统开机后启动后的第一个进程，有它来执行其他进程的启动。比如一些被配置为开机自启动的服务。

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220515225627398.png" alt="image-20220515225627398" style="zoom:50%;" />

*tips：运行级别**1**用于忘记密码进行密码的重置。原理是系统文件目录的重新挂载，能够把只读的目录设置为可读可写。

systemctl get-default		查看当前运行级别

init n		设置运行级别



#### 关机重启

shutdown -c -h [n] [time_format]

关机前要先把磁盘和内存数据做一次sync，所以默认关机是需要时间来酝酿的

reboot = shutdown -r now

halt		停机不断电，内存数据得以保留

poweroff		掐断电源不做同步



### 进程管理

---

pid唯一标识一个进程

常驻内存的是**服务**，正在执行的叫做**进程**

以d结尾的服务一般都是守护进程，比如mysqld.service

ps aux | grep xxx		查看系统中的所有进程（xxx也可以是命令，比如less或more）

ps -ef | grep xxx		查看子父进程之间的关系

 ps		当前用户和终端执行的进程（很少比较局限不代表全部）

 

![image-20220516112123913](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220516112123913.png)



![image-20220516112451383](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220516112451383.png)

VSZ 虚拟内存占用量

RSS 物理内存的占用量

TTY 调用该进程的终端

COMMAND 调用该进程的命令

STAT 进程状态；R 运行 S 睡眠 T 暂停 Z 僵尸 s 包含子进程 l 多线程 + 前台显示

TIME 该进程占用CPU的运算时间

Linux的内存页面置换通常使用LRU算法

kill -9 [pid]		立即杀死进程

pstree  -p 显示pid -u 显示进程所属用户



#### **top		实时系统监控**

![image-20220516132850138](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220516132850138.png)

每3s刷新一次

load average 前中后三个数值分别表示1分钟、3分钟、5分钟前系统的负载

us 没有更改优先级的进程 | sy 系统进程 | ni 用户设置优先级的进程 | id 空闲 | wa 等待IO的进程 | 

hi 硬件中断 | si 软件中断 | st stolen 被虚拟化的进程占比

PR 优先级 | NI 用户指定的nice值 | VIRT 虚拟内存用量 | RES 物理内存用量 | SHR 共享内存用量 | S 运行状态

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220516134151673.png" alt="image-20220516134151673" style="zoom:50%;" />





### 网络管理

---

netstat -anp | grep pid		查看该进程的网络信息

netstat -nlp | grep port-id		查看网络端口号占用情况



![image-20220516151835724](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20220516151835724.png)





### 面试篇

---



#### 问题排查步骤

##### 使用top查看系统的负载

不仅仅需要查看每个进程占用情况，也需要在综合栏上看系统总体负载和每个CPU的负载，快速定位一下系统资源的消耗情况

uptime快速查看系统的overall负载

load average平均超过了60%说明系统的负载较高

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230223110348456.png" alt="image-20230223110348456" style="zoom: 33%;" />



##### vmstat查看CPU资源占用情况

r - running

b - blocked

us - user

sy - system

us + sy 大于80%说明系统的负载较高

进程数不宜超过2个/核

![image-20230223111108776](https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230223111108776.png)



##### mpstat -P ALL 2 查看机器上的全部CPU的使用情况（只有消耗而没有具体的进程信息）



##### pidstat -u 1 -p 5101

根据pid查看具体进程的CPU占用情况



##### ps -ef | grep java

详细输出java相关进程的相关信息 -ef可以理解为详细信息；这一步的意义通常是为了定位到具体的问题线程



##### free -[kb/m/g] 查看内存占用情况

参数是统计的单位，MB最为推荐，精细地刚刚好



##### pidstat -p [pid] -r [sampling interval] 

查看具体线程的内存占用情况



##### df -h 查看磁盘使用情况



##### iostat pidstat -p [pid] -r [sampling interval] 查看磁盘IO占用情况

磁盘IO是拖慢系统效率的一大重要元素



##### ifstat -1 查看网络IO情况

通常有docker这种虚拟机也会显示出来



#### 定位生产问题的步骤（性能相关的问题）

根据上面的一番检查后我们大致能知道是什么进程的什么线程导致了系统过大的负载

这个时候在Java领域当中我们应该启用jstack来对具体的线程进行一个分析

**jstack [pid] | grep "tid"** 这行命令会输出分析看看这个Java进程里哪个线程过载

最好是 jstack [pid] >> xxx.log 输出到日志文件中去分析

##### 

<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230223120249175.png" alt="image-20230223120249175" style="zoom: 25%;" />



<img src="https://hansomehu-picgo.oss-cn-hangzhou.aliyuncs.com/typora/image-20230223120358219.png" alt="image-20230223120358219" style="zoom:25%;" />

这是一个jstack的案例

```shell
Found one Java-level deadlock:

"Thread-1":
waiting to lock monitor 0x00007f0134003ae8 (object 0x00000007d6aa2c98, a java.lang.Object),
which is held by "Thread-0""Thread-0":
waiting to lock monitor 0x00007f0134006168 (object 0x00000007d6aa2ca8, a java.lang.Object),
which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
at javaCommand.DeadLockclass.run(JStackDemo.java:40)
- waiting to lock <0x00000007d6aa2c98> (a java.lang.Object)
- locked <0x00000007d6aa2ca8> (a java.lang.Object)
at java.lang.Thread.run(Thread.java:745)
"Thread-0":
at javaCommand.DeadLockclass.run(JStackDemo.java:27)
- waiting to lock <0x00000007d6aa2ca8> (a java.lang.Object)
- locked <0x00000007d6aa2c98> (a java.lang.Object)
at java.lang.Thread.run(Thread.java:745)
```



GC方面的问题分析，一般都是参数开启GCPrintDetails，然后把文件dump到log文件中

之后将日志文件使用类似GCViewer等工具进行可视化分析，重点来看GC的频率

```shell
-XX:+PrintGC 

-XX:+PrintGCDetails 

-Xloggc:gc.log 

#### 具体GC文件的例子
[GC (CMS Initial Mark) [1 CMS-initial-mark: 19498K(32768K)] 36184K(62272K), 0.0018083 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
[CMS-concurrent-mark-start]
[CMS-concurrent-mark: 0.011/0.011 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
[CMS-concurrent-preclean-start]
[CMS-concurrent-preclean: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[CMS-concurrent-abortable-preclean-start]
 CMS: abort preclean due to time [CMS-concurrent-abortable-preclean: 0.558/5.093 secs] [Times: user=0.57 sys=0.00, real=5.09 secs] 
[GC (CMS Final Remark) [YG occupancy: 16817 K (29504 K)][Rescan (parallel) , 0.0021918 secs][weak refs processing, 0.0000245 secs][class unloading, 0.0044098 secs][scrub symbol table, 0.0029752 secs][scrub string table, 0.0006820 secs][1 CMS-remark: 19498K(32768K)] 36316K(62272K), 0.0104997 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
[CMS-concurrent-sweep-start]
[CMS-concurrent-sweep: 0.007/0.007 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
[CMS-concurrent-reset-start]
[CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
```





jmap命令行工具的使用，如果jstack的分析不够用的话，**jmap**能对内存使用、泄露来进行更加深入的分析.jmap命令是一个可以输出所有内存中对象的工具，甚至可以将VM 中的heap



jmap [pid]



jmap -heap [pid]

显示Java堆详细信息



jmap -histo:live [pid]
显示堆中对象的统计信息



jmap -finalizerinfo [pid]

打印等待终结的对象信息



jmap -clstats [pid]
打印类加载器信息



