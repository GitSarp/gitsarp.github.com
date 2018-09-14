---
title: JVM命令
date: 2018-07-11 22:08:08
tags: JVM
categories: Java
---
#### 常用java命令
1. jps——查看虚拟机进程信息
jps [options]<hostid>
![](/images/jps.png)
- 第一列是VMpid，第二列class或jar的名称
- hostid指定主机，不写为当前机器，格式   [protocol:]<<//>hostname><:port></servername>
- q：仅显示lvmid；
- **-m**:输出主函数传入的参数；
- **-l**: 输出应用程序主类完整package名称或jar完整名称；
- **-v**: 输出jvm启动时指定的jvm参数；
- -V: 输出通过.hotsportrc或-XX:Flags=<filename>指定的jvm参数

2.jstat——显示虚拟机运行时状态信息
jstat [options] <lvmid> [intervals[s|ms]] [count]
intervals：输出间隔时间
count：输出总次数
option参数：
- class 加载class的数量，字节大小和未加载的class数量大小以及加载时间
 $ jstat -class 11589
 Loaded  Bytes  Unloaded  Bytes     Time   
  7035  14506.3     0     0.0       3.67
- compiler HotSpot JIT编译器行为统计
 $ jstat -compiler 1262
Compiled Failed Invalid   Time   FailedType FailedMethod
    2573      1       0    47.60          1 org/apache/catalina/loader/WebappClassLoader findResourceInternal
- gc 分别输出survivoro、Eden和老年代的空间和已用空间大小，以及年轻代和老年代gc的次数，耗时以及gc总耗时
$ jstat -gc 1262
 S0C    S1C     S0U     S1U   EC       EU        OC         OU        PC       PU         YGC    YGCT    FGC    FGCT     GCT   
26112.0 24064.0 6562.5  0.0   564224.0 76274.5   434176.0   388518.3  524288.0 42724.7    320    6.417   1      0.398    6.815

- gccapacity 除上条输出信息外，还输出每个堆区域的最小（大）空间限制
$ jstat -gccapacity 1262
 NGCMN    NGCMX     NGC    S0C   S1C       EC         OGCMN      OGCMX      OGC        OC       PGCMN    PGCMX     PGC      PC         YGC    FGC 
614400.0 614400.0 614400.0 26112.0 24064.0 564224.0   434176.0   434176.0   434176.0   434176.0 524288.0 1048576.0 524288.0 524288.0    320     1
- gccasue 同gcutil，和最后一次执行gc的原因以及当前执行gc的原因
$ jstat -gccause 28920
  S0     S1     E      O      P       YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC                 
 12.45   0.00  33.85   0.00   4.44      4    0.242     0    0.000    0.242   Allocation Failure   No GC
- gcnew 输出新生代空间的gc性能数据
- gcnewcapacity 新生代空间大小的统计数据
- gcold 老年代空间的gc性能数据
- gcoldcapacity 老年代空间大小的统计数据
- gcpermcapacity 持久代的空间大小统计数据
- gcutil 同gc，只不过输出的是已使用空间占总空间的百分比
$ jstat -gcutil 28920
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT   
 12.45   0.00  33.85   0.00   4.44  4       0.242     0    0.000    0.242
- printcompilation HotSpot编译方法统计

3. jstack——查看线程调用栈信息
jstack [option] LVMID
- LVMID:jps命令，或将占用cpu或其他资源多的pid找出来(可以通过top命令)，然后转成16进制的数，即为lvmid
- *-F* : 当正常输出请求不被响应时，强制输出线程堆栈
- **-l** : 除堆栈外，显示关于锁的附加信息
- **-m** : 如果调用到本地方法的话，可以显示C/C++的堆栈
![](https://upload-images.jianshu.io/upload_images/2184951-311ab1b4ea7dde3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
4. jmap [ -option ] <lvmid>--生成heap dump文件
- dump,-dump[:live,]format=b,file=<filename>,live说明是否只dump存活的对象.或者使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候·自动生成dump文件
- heap 显示堆详细信息，如使用哪种回收器、参数配置、分代情况等

```
$ jmap -heap 28920
  Attaching to process ID 28920, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 24.71-b01  
 
  using thread-local object allocation.
  Parallel GC with 4 thread(s)//GC 方式  
 
  Heap Configuration: //堆内存初始化配置
     MinHeapFreeRatio = 0 //对应jvm启动参数-XX:MinHeapFreeRatio设置JVM堆最小空闲比率(default 40)
     MaxHeapFreeRatio = 100 //对应jvm启动参数 -XX:MaxHeapFreeRatio设置JVM堆最大空闲比率(default 70)
     MaxHeapSize      = 2082471936 (1986.0MB) //对应jvm启动参数-XX:MaxHeapSize=设置JVM堆的最大大小
     NewSize          = 1310720 (1.25MB)//对应jvm启动参数-XX:NewSize=设置JVM堆的‘新生代’的默认大小
     MaxNewSize       = 17592186044415 MB//对应jvm启动参数-XX:MaxNewSize=设置JVM堆的‘新生代’的最大大小
     OldSize          = 5439488 (5.1875MB)//对应jvm启动参数-XX:OldSize=<value>:设置JVM堆的‘老生代’的大小
     NewRatio         = 2 //对应jvm启动参数-XX:NewRatio=:‘新生代’和‘老生代’的大小比率
     SurvivorRatio    = 8 //对应jvm启动参数-XX:SurvivorRatio=设置年轻代中Eden区与Survivor区的大小比值 
     PermSize         = 21757952 (20.75MB)  //对应jvm启动参数-XX:PermSize=<value>:设置JVM堆的‘永生代’的初始大小
     MaxPermSize      = 85983232 (82.0MB)//对应jvm启动参数-XX:MaxPermSize=<value>:设置JVM堆的‘永生代’的最大大小
     G1HeapRegionSize = 0 (0.0MB)  
 
  Heap Usage://堆内存使用情况
  PS Young Generation
  Eden Space://Eden区内存分布
     capacity = 33030144 (31.5MB)//Eden区总容量
     used     = 1524040 (1.4534378051757812MB)  //Eden区已使用
     free     = 31506104 (30.04656219482422MB)  //Eden区剩余容量
     4.614088270399305% used //Eden区使用比率
  From Space:  //其中一个Survivor区的内存分布
     capacity = 5242880 (5.0MB)
     used     = 0 (0.0MB)
     free     = 5242880 (5.0MB)
     0.0% used
  To Space:  //另一个Survivor区的内存分布
     capacity = 5242880 (5.0MB)
     used     = 0 (0.0MB)
     free     = 5242880 (5.0MB)
     0.0% used
  PS Old Generation //当前的Old区内存分布
     capacity = 86507520 (82.5MB)
     used     = 0 (0.0MB)
     free     = 86507520 (82.5MB)
     0.0% used
  PS Perm Generation//当前的 “永生代” 内存分布
     capacity = 22020096 (21.0MB)
     used     = 2496528 (2.3808746337890625MB)
     free     = 19523568 (18.619125366210938MB)
     11.337498256138392% used  
 
  670 interned Strings occupying 43720 bytes.
```
- finalizerinfo 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
- histo 显示堆中对象统计信息，包括类，实例数量和合计数量

```
$ jmap -histo:live 28920 | more
 num     #instances         #bytes  class name
----------------------------------------------
   1:         83613       12012248  <constMethodKlass>
   2:         23868       11450280  [B
   3:         83613       10716064  <methodKlass>
   4:         76287       10412128  [C
   5:          8227        9021176  <constantPoolKlass>
   6:          8227        5830256  <instanceKlassKlass>
   7:          7031        5156480  <constantPoolCacheKlass>
   8:         73627        1767048  java.lang.String
   9:          2260        1348848  <methodDataKlass>
  10:          8856         849296  java.lang.Class
```
class name:
B  byte
C  char
D  double
F  float
I  int
J  long
Z  boolean
[  数组，如[I表示int[]
[L+类名 其他对象

- permstat 打印永久代统计信息
- 
```
$ jmap -permstat 28920
  Attaching to process ID 28920, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 24.71-b01
  finding class loader instances ..done.
  computing per loader stat ..done.
  please wait.. computing liveness.liveness analysis may be inaccurate ...
 
  class_loader            classes bytes   parent_loader           alive?  type  
  <bootstrap>             3111    18154296          null          live    <internal>
  0x0000000600905cf8      1       1888    0x0000000600087f08      dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
  0x00000006008fcb48      1       1888    0x0000000600087f08      dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
  0x00000006016db798      0       0       0x00000006008d3fc0      dead    java/util/ResourceBundle$RBClassLoader@0x0000000780626ec0
  0x00000006008d6810      1       3056      null          dead    sun/reflect/DelegatingClassLoader@0x00000007800500a0
```
- F当-dump没有响应时，强制生成dump快照
5. jhat(JVM Heap Analysis Tool)，用来分析jmap 生成的dump(耗费资源，一般将服务器的dump拷贝下来分析，MAT分析比jhat更好点)
jhat [dumpfile]
$ jhat -J-Xmx512m dump.hprof
浏览器访问Http://localhost:7000即可
6. jinfo   jps -v口令只能查看到显式指定的参数，如果想要查看未被显式指定的参数的值
jinfo [option] [args] LVMID
-flag : 输出指定args参数的值
-flags : 不需要args参数，输出所有JVM参数的值
-sysprops : 输出系统属性，等同于System.getProperties()
7. jdps
jdps <options> <classses...>
用来分析.class文件,目录或jar文件的路径名
>jdps -R -doutput ~/tmp HelloWorld.class
>jdps -p java.lang xxx.jar
查看jar包的所有package(模块信息)
>jdps --inverse --require java.lang(jdk9以上)
查看当前目录和环境路径下那些包依赖指定的包
### JVM参数配置分析

配置文件位置：eclipse.ini、TOMCAT_HOME/bin/catalina.sh（.bat）、idea有-vm option

[](http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html)
