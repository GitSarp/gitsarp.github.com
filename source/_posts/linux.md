---
title: 关于Linux
date: 2018-07-01 21:31:48
tags: Linux
categories: Linux
---
##  性能查看命令
#### java相关
ps -ef | grep java 				找所有有关“java”的进程
ps -efL | grep [PID] | wc -l 		查看某个进程创建的线程数
jmap -histo [pid] 				按照对象内存大小排序 注意会导致full gc
#### 内存
1. free
```
free [－b　－k　－m][－o] [－s delay][－t] [－V]
－b －k －m：分别以字节（KB、MB）为单位显示内存使用情况。
－s delay：显示每隔多少秒数来显示一次内存使用情况。
－t：显示内存总和列。 －o：不显示缓冲区调节列。
和top命令相比，它的优点是使用简单，并且只占用很少的系统资源。
```
2. vmstat
```
vmstat 3(间隔时间) 100(监控次数)
eg:
[root@iZbp10fstdmbee21wif5ngZ 2018-02-26]# vmstat 3 100
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
r b swpd free buff cache si so bi bo in cs us sy id wa st
 1 0 0 399392 220280 6405804 0 0 1 4 1 2 1 0 98 0 0
 0 0 0 398648 220280 6405960 0 0 0 25 2725 2284 2 0 98 0 0
 0 0 0 398648 220280 6406036 0 0 0 91 2652 2255 2 0 97 0 0
 0 0 0 398384 220280 6406080 0 0 0 112 2539 2180 2 0 98 0 0
 1 0 0 398352 220280 6406184 0 0 0 0 2724 2230 2 0 98 0 0
 
 r：等待在CPU资源的进程数。这个数据比平均负载更加能够体现CPU负载情况。如果这个数值大于机器CPU核数，那么机器的CPU资源已经饱和。
si, so：交换区写入和读取的数量。如果这个数据不为0，说明系统已经在使用交换区（swap），机器物理内存已经不足。
us, sy, id, wa, st：这些都代表了CPU时间的消耗，它们分别表示用户时间（user）、系统（内核）时间（sys）、空闲时间（idle）、IO等待时间（wait）和被偷走的时间（stolen，一般被其他虚拟机消耗）。
一般情况下，如果用户时间和系统时间相加非常大，CPU处于忙于执行指令。如果IO等待时间很长，那么系统的瓶颈可能在磁盘IO。
 
```
3. top
#### 网络
1. sar
```
sar -n { DEV | EDEV | NFS | NFSD | SOCK | ALL }
DEV显示网络接口信息，
EDEV显示关于网络错误的统计数据，
NFS统计活动的NFS客户端的信息，
NFSD统计NFS服务器的信息，
SOCK显示套接字信息，
ALL显示所有5个开关

[root@iZbp10fstdmbee21wif5ngZ 2018-02-26]# sar -n SOCK 2 10
Linux 3.10.0-514.6.2.el7.x86_64 (iZbp10fstdmbee21wif5ngZ) 02/27/2018 _x86_64_	(8 CPU)
05:07:33 PM totsck tcpsck udpsck rawsck ip-frag tcp-tw
05:07:35 PM 265 163 4 0 0 12
05:07:37 PM 265 163 4 0 0 12
totsck:使用的套接字总数量
tcpsck:使用的TCP套接字数量
udpsck:使用的UDP套接字数量
rawsck:使用的raw套接字数量
ip-frag:使用的IP段数量

[root@iZbp10fstdmbee21wif5ngZ 2018-02-26]# sar -n DEV 2 10
Linux 3.10.0-514.6.2.el7.x86_64 (iZbp10fstdmbee21wif5ngZ) 02/27/2018 _x86_64_	(8 CPU)
05:12:43 PM IFACE rxpck/s txpck/s rxkB/s txkB/s rxcmp/s txcmp/s rxmcst/s
05:12:45 PM eth0 535.50 397.00 58.26 67.17 0.00 0.00 0.00
05:12:45 PM eth1 11.50 13.00 4.18 2.49 0.00 0.00 0.00
05:12:45 PM lo 1.50 1.50 0.09 0.09 0.00 0.00 0.00
IFACE：LAN接口
rxpck/s：每秒钟接收的数据包
txpck/s：每秒钟发送的数据包
rxbyt/s：每秒钟接收的字节数
txbyt/s：每秒钟发送的字节数
rxcmp/s：每秒钟接收的压缩数据包
txcmp/s：每秒钟发送的压缩数据包
rxmcst/s：每秒钟接收的多播数据包

[root@iZbp10fstdmbee21wif5ngZ 2018-02-26]# sar -n EDEV 2 10
Linux 3.10.0-514.6.2.el7.x86_64 (iZbp10fstdmbee21wif5ngZ) 02/27/2018 _x86_64_	(8 CPU)
05:13:47 PM IFACE rxerr/s txerr/s coll/s rxdrop/s txdrop/s txcarr/s rxfram/s rxfifo/s txfifo/s
05:13:49 PM eth0 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
05:13:49 PM eth1 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00 0.00
IFACE：LAN接口
rxerr/s：每秒钟接收的坏数据包
txerr/s：每秒钟发送的坏数据包
coll/s：每秒冲突数
rxdrop/s：因为缓冲充满，每秒钟丢弃的已接收数据包数
txdrop/s：因为缓冲充满，每秒钟丢弃的已发送数据包数
txcarr/s：发送数据包时，每秒载波错误数
rxfram/s：每秒接收数据包的帧对齐错误数
rxfifo/s：接收的数据包每秒FIFO过速的错误数
txfifo/s：发送的数据包每秒FIFO过速的错误数
```
#### 磁盘
1. iostat(具体查看manual)
```
eg.
iostat -d -k 1 10 #查看TPS和吞吐量信息
iostat -d -x -k 1 10 #查看设备使用率（%util）、响应时间（await）
iostat -c 1 10 #查看cpu状态
[root@iZbp10fstdmbee21wif5ngZ 2018-02-26]# iostat -d -x -k 1 10
Linux 3.10.0-514.6.2.el7.x86_64 (iZbp10fstdmbee21wif5ngZ) 02/27/2018 _x86_64_	(8 CPU)
Device: rrqm/s wrqm/s r/s w/s rkB/s wkB/s avgrq-sz avgqu-sz await r_await w_await svctm %util
vda 0.01 4.32 0.51 2.99 18.01 32.07 28.56 0.04 11.71 60.41 3.34 0.73 0.26
```
## 其他命令
1. grep
```
参数：
  -R 递归
  -w 按单词查找
  -l 列出文件名
  -i 忽略大小写
  --exclude=　排除
  --exclude-dir　排除目录
  -n　输出行号
  -v   反向，即不包含
eg.搜索内容包含指定字符串的文件
  grep -s xxx /etc/*
  grep -R xxx /etc/*        包含子目录递归搜索
  grep -Rw xxx /etc/*      w-不会查找出包含xxxy的
  grep -Rl xxx /etc/*        只输出文件名
  grep -Ril xxx /etc/*        忽略大小写
  grep -Ril xxx /etc/*.conf
  grep -Ril --exclude=\*.conf xxx /etc/*   .conf文件不查找
  grep --exclude-dir=/etc/grub.d -Rwl xxx /etc/*  排除/etc/grub.d目录
  grep -Rni xxx /etc/*.conf n-输出字符串所在行行号
  grep -Rlv xxx /etc/*  查找出所有不包含xxx的文件
```
## VIM

除了基本的会计键外，可以通过修改vim的配置文件实现编译运行程序(在 VimScript 中函数名必须以大写字母开头，否则 Vim 将提示错误；exec 关键字之后的都会在Vim 的命令模式下执行)

![create functions in vim .vimrc](http://upload-images.jianshu.io/upload_images/3341325-948dfd8baa0cb1b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
类似地：
![create function in vim for python](http://upload-images.jianshu.io/upload_images/3341325-9535fd7a41a09db2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![create function in vim for java](http://upload-images.jianshu.io/upload_images/3341325-49e59bb7f5b83f88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
autocmd 实现这个，执行不同程序设计语言编译出的代码。
![autocmd in vimrc](http://upload-images.jianshu.io/upload_images/3341325-962de92314dd0be0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 发行版

 Ubuntu，已经不用了，经常会有bug和包的冲突，可定制性也不高。
> 爱上archlinux的原因

  - 档案安装Wiki一步步装完archlinux的时候，你会惊讶于这个OS的勇气，始终坚持Keep simple的理念。呈现在你面前的只有一个虚拟终端。这其实意味着你需要付出更多的努力，但是却能保证打造出你最喜欢的系统，真正属于你的。

  - 如果你还不够厉害（just like me），安装显卡驱动和xorg，再配上一个awesome或者其他的窗口管理器，就可以开始你的探险之旅了。
> 推荐给你的软件

- vimfx 一款用vim方式操作firefox（pacman -S flashplugin安装flash）的插件。
- zsh 一款好用的终端。
- awesome主题 [awesome-copycats](https://github.com/copycat-killer/awesome-copycats)等。
>未完待续


### 问题记录：
#### 1.bios中没有arch引导项或选择arch后，grub 进入rescue模式，提示unknown file system
>可以通过easyUEFI，管理efi启动项

![选择esp启动分区](http://upload-images.jianshu.io/upload_images/3341325-1a30e678250ff848.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![选择grubx64.efi所在](http://upload-images.jianshu.io/upload_images/3341325-96c3f550011e16c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>在grub rescue模式中，也可以通过输入以下命令进入Linux系统
- ls
- ls 各个分区，找到文件系统为fat的启动分区，unknown即不是（diskgenius辅助）
- set root=(hd1，gpt5)     设置启动分区
- prefix= /grub  （x86_64-efi所在的目录）
- insmod normal
以上步骤没错的话，输入以下命令可进入grub菜单，否则是这两个参数有错
- normal

- 按e编辑启动项为刚刚的分区，ctrl+x引导，最后进入系统后修改grub.cfg(用grub-mkconfig -o /boot/grub/grub.cfg自动生成)





