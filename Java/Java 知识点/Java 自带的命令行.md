# Jdk 自带的命令行

## jps

作用：查看当前系统中的 java 进程

```bash
jps [-q] [-mlvV] [<hostid>]
```

>1. -q : 只输出进程 ID
>2. -m : 输出传入 main 方法的参数
>3. -l : 输出完全的包名，应用主类名，jar的完全路径名
>4. -v : 输出jvm参数
>5. -V : 输出通过flag文件传递到JVM中的参数(.hotspotrc文件或-XX:Flags=所指定的文件)


**jps 远程查看**  
jps 支持远程调用，如：`jps 192.168.2.83`。但是需要在被查看的服务器开启 jstatd 服务。

步骤：
>1. 创建 jstatd.all.policy 策略文件
```java
grant codebase "file:${java.home}/../lib/tools.jar" {
    permission java.security.AllPermission;
};
```

>2. 启动 jstatd 服务
```bash
jstatd -J-Djava.security.policy=jstatd.all.policy -J-Djava.rmi.server.hostname=192.168.31.241
```

-J 参数是一个公共的参数，如 jps、 jstat 等命令都可以接收这个参数。由于 jps、 jstat 命令本身也是 Java 应用程序， -J 参数可以为 jps 等命令本身设置 Java 虚拟机参数。

-Djava.security.policy：指定策略文件  
-Djava.rmi.server.hostname：指定服务器的ip地址（可忽略）


## jstack

作用
>1. 查看 java 程序中当前时刻的线程堆栈信息
>2. 针对 core 文件分析线程堆栈信息

```bash
jstack [-l] <pid>
    (to connect to running process)
jstack -F [-m] [-l] <pid>
    (to connect to a hung process)
jstack [-m] [-l] <executable> <core>
    (to connect to a core file)
jstack [-m] [-l] [server_id@]<remote server IP or hostname>
    (to connect to a remote debug server)
```

>1. -l : 打印关于锁的附加信息,例如属于java.util.concurrent 的 ownable synchronizers列表。会使得JVM停顿得长久得多，不建议使用
>2. -F : 强制打印栈信息
>3. -m : 打印 java 和 native c/c++ 框架的所有栈信息

使用：
![Alt 'jstack'](https://s2.ax1x.com/2020/01/09/lfYKW6.png)  
通过图片我们可以知道：当前 JVM 所有线程的状态，线程Id，锁情况等，通过分析他们的情况，我们可以了解到死锁等问题。

当然的 jstack 也支持远程调用，用法和 jsp 类似(没有实测过)


## jmap
作用
>1. 生成 Java 程序的 Heap Dump 文件
>2. 当前 JVM 的堆情况

```bash
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```

从上面可知道，jmap 有三种使用情况
>1. 针对当前系统内的某个进程
>2. 针对一个 core 文件
>3. 针对远程的某个程序

指定的参数进行使用

>1. 没有任何参数 `jmap 进程Id`  
打印虚拟机中加载的每个共享对象的起始地址、映射大小以及共享对象文件的路径全称

```bash
Attaching to process ID 11340, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.162-b12
0x0000000000400000  7K  /opt/jdk-8u162/bin/java
0x00007fa6bc1b9000  91K /opt/jdk-8u162/jre/lib/amd64/libnio.so
0x00007fa6bcbca000  66K /usr/lib64/libbz2.so.1.0.6
0x00007fa6bcdda000  153K    /usr/lib64/liblzma.so.5.2.2
0x00007fa6d007d000  88K /usr/lib64/libz.so.1.2.7

...
```

>2. heap 参数 `jmap -heap 进程Id`  
堆的摘要信息，包括使用的GC算法、堆配置信息和各内存区域内存使用信息

```bash
Attaching to process ID 11340, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.162-b12

using thread-local object allocation.
Mark Sweep Compact GC

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   
...
```

>3. -histo:[live] 参数 `jmap -histo:live 进程Id`  
显示堆中对象的统计信息，包括对象数量、内存大小(单位：字节)、完全限定的类名。如果指定了live子选项，则只计算活动的对象。

```bash
#num       #instance        #byte     class name

1:             2             32  org.springframework.core.convert.support.ObjectToObjectConverter
2:             2             32  org.springframework.core.convert.support.ObjectToOptionalConverter

...
```

>4. -clstats 参数 `jmap -clstats 进程Id`  
堆内存的永久保存区域的类加载器的智能统计信息，包括它的名称、活跃度、地址、父类加载器、它所加载的类的数量和大小

```bash
class_loader    classes bytes   parent_loader   alive?  type

<bootstrap> 1640    2901997   null      live    <internal>
0x00000000ed186018  1   1471    0x00000000ecdbaeb0  dead    sun/reflect/DelegatingClassLoader@0x0000000100009df8

...
```

>5. finalizerinfo 参数  `jmap -finalizerinfo 进程Id`  
打印等待终结的对象信息

```bash
Attaching to process ID 11340, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.162-b12
Number of objects pending for finalization: 0

...
```

>6. dump:<dump-options> `jmap -dump:format=b,file=heapdump.phrof 进程Id`  
将这个JVM 当前的堆信息转为文件的信息。这个过程是很长的，同时为了收集数据的准确性，在执行这个命令的过程中，JVM 会暂停当前的应用，所以生产谨慎使用
```bash
Dumping heap to /root/heapdump.phrof ...
Heap dump file created
```

## jinfo
作用：
>1. 查看当前程序的扩展参数，包括 Java System 属性和 JVM 命令行参数
>2. 动态的修改正在运行的 JVM 一些参数
>3. 分析 core 文件

```bash
Usage:
    jinfo [option] <pid>
        (to connect to running process)
    jinfo [option] <executable <core>
        (to connect to a core file)
    jinfo [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    -flag <name>         to print the value of the named VM flag
    -flag [+|-]<name>    to enable or disable the named VM flag
    -flag <name>=<value> to set the named VM flag to the given value
    -flags               to print VM flags
    -sysprops            to print Java system properties
    <no option>          to print both of the above
    -h | -help           to print this help message
```

使用情景有 3 个
>1. 真的当前的程序
>2. 针对 core 文件
>3. 远程的程序

参数说明
>1. no option 输出全部的参数和系统属性
>2. -flag name 输出对应名称的参数, 可以查看指定的 jvm 参数的值
>3. -flag [+|-]name 开启或者关闭对应名称的参数, 动态的修改 jvm 的参数(只可以对那些 boolean 情况的参数使用，既只需要指定这个参数，不需要指定这个参数的值的情况)
>4. -flag name=value 设定对应名称的参数 (同情况三差不多，不过这个使用在 key = value 的情况)
>5. -flags 输出全部的参数
>6. -sysprops 输出系统属性

虽然支持动态的修改，但是并不是所有的参数都支持动态修改


## jcmd
作用：
>1. 导出堆
>2. 查看Java进程
>3. 导出线程信息
>4. 执行GC
>5. 还可以进行采样分析

等等，可以说是一个具备已有命令行所有功能的命令行了

```bash
Usage: jcmd <pid | main class> <command ...|PerfCounter.print|-f file>
   or: jcmd -l                                                    
   or: jcmd -h                                                    
                                                                  
  command must be a valid jcmd command for the selected jvm.      
  Use the command "help" to see which commands are available.   
  If the pid is 0, commands will be sent to all Java processes.   
  The main class argument will be used to match (either partially 
  or fully) the class used to start Java.                         
  If no options are given, lists Java processes (same as -p).     
                                                                  
  PerfCounter.print display the counters exposed by this process  
  -f  read and execute commands from the file                     
  -l  list JVM processes on the local machine                     
  -h  this help      
```

参数说明:
>1. <pid | main class>： jcmd 除了可以通过进程 Id 进行操作外，还可以通过 指定程序的 main class 进程操作
>2. command : 诊断命令，发送给所有的请求给指定的程序，多个陈序的 main class 一样，都会接受到
>3. Perfcounter.print：打印目标Java进程上可用的性能计数器
>4. -f file：从文件file中读取命令，然后在目标Java进程上调用这些命令
>5. -l：查看所有的进程列表信息

使用
>1. `jcmd -l`: 查看当前系统的所有 java 进程，效果等同于 `jps`

>2. `jcmd 进程Id PerfCounter.print`: 查看指定进程的性能统计信息

```bash
java.ci.totalTime=8333450794
java.cls.loadedClasses=5874
java.cls.sharedLoadedClasses=0
java.cls.sharedUnloadedClasses=0
java.cls.unloadedClasses=0
java.property.java.class.path="demo-0.0.1-SNAPSHOT.jar"
java.property.java.endorsed.dirs="/opt/jdk-8u162/jre/lib/endorsed"
...
```

>3. `jcmd 进程Id help`: 列出当前运行的 java 进程可以执行的操作

```bash
The following commands are available:
JFR.stop
JFR.start
JFR.dump
JFR.check
VM.native_memory
VM.check_commercial_features
VM.unlock_commercial_features
ManagementAgent.stop
ManagementAgent.start_local
ManagementAgent.start
GC.rotate_log
Thread.print
GC.class_stats
GC.class_histogram
GC.heap_dump
GC.run_finalization
GC.run
VM.uptime
VM.flags
VM.system_properties
VM.command_line
VM.version
help
```

>4. `jcmd 进程Id help 命令名`：查看具体命令的选项

```bash
JFR.stop
Stops a JFR recording

Impact: Low

Permission: java.lang.management.ManagementPermission(monitor)

Syntax : JFR.stop [options]

Options: (options must be specified using the <key> or <key>=<value> syntax)
    name : [optional] Recording name,.e.g \"My Recording\" (STRING, no default value)
    recording : [optional] Recording number, see JFR.check for a list of available recordings (JLONG, -1)
    discard : [optional] Skip writing data to previously specified file (if any) (BOOLEAN, false)
    filename : [optional] Copy recording data to file, e.g. \"/home/user/My Recording.jfr\" (STRING, no default value)
    compress : [optional] GZip-compress "filename" destination (BOOLEAN, false)
```

>5. `jcmd 进程Id 命令名=参数值,命令名=参数名`: 执行命令(有些命令只起查看功能的，可以不用参数)

```bash
>jcmd 13383 VM.version(查看虚拟机版本)
Java HotSpot(TM) 64-Bit Server VM version 25.162-b12
JDK 8.0_162
```
其他的参数就不试了，具体的功能也不太清楚...


## jstat
作用：输出程序中的堆内存的各种情况，如堆大小，加载类数量，垃圾回收情况, 各个区域的使用情况等

```bash
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as 
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.

option:
-class
-compiler
-gc
-gccapacity
-gccause
-gcmetacapacity
-gcnew
-gcnewcapacity
-gcold
-gcoldcapacity
-gcutil
-printcompilation
```

参数：  
t：在打印的列加上Timestamp列，用于显示系统运行的时间  
h: 在在周期性打印数据情况的时候，指定输出多少行以后输出一次表头  
vmid： Virtual Machine ID（ 进程的 pid）  
interval： 执行间隔，单位毫秒  
count： 指定输出多少次记录，缺省则会一直打印

使用情景
>1. `jstat -class 进程Id`: 显示 ClassLoad 的相关信息  

```bash
Loaded  Bytes  Unloaded  Bytes     Time   
  5874 10696.2        0     0.0       5.49
```
[注]  
Loaded: 加载的类数量  
Bytes: 占用空间的大小  
Unloaded: 已经卸载类的数量  
Time：装载和卸载类所花费的时间

>2. `jstat -compiler 进程Id`: 显示VM实时编译(JIT)的数量等信息

```bash
Compiled Failed Invalid   Time   FailedType FailedMethod
    2564      0       0     8.77          0 
```
[注]  
Compiled：编译任务执行数量  
Failed：编译任务执行失败数量  
Invalid：编译任务执行失效数量  
Time：编译任务消耗时间  
FailedType：最后一个编译失败任务的类型  
FailedMethod：最后一个编译失败任务所在的类及方法  

>3. `jstat -gc 进程Id`: 显示gc相关的堆信息，查看gc的次数，及时间

```bash
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
1024.0 1024.0  0.0   538.9   8256.0   3753.9   20480.0    10523.1   31360.0 29641.5 4224.0 3871.4     43    0.213   1      0.035    0.248
```
[注]  
S0C：年轻代中第一个 survivor（幸存区）的容量 （字节）  
S1C：年轻代中第二个 survivor（幸存区）的容量 (字节)  
S0U：年轻代中第一个 survivor（幸存区）目前已使用空间 (字节)  
S1U：年轻代中第二个 survivor（幸存区）目前已使用空间 (字节)  
EC：年轻代中 Eden（伊甸园）的容量 (字节)  
EU：年轻代中 Eden（伊甸园）目前已使用空间 (字节)  
OC：old 代的容量 (字节)  
OU：old 代目前已使用空间 (字节)  
MC：metaspace (元空间)的容量 (字节)  
MU：metaspace (元空间)目前已使用空间 (字节)  
YGC：从应用程序启动到采样时年轻代中 gc 次数  
YGCT：从应用程序启动到采样时年轻代中 gc 所用时间(s)  
FGC：从应用程序启动到采样时 old 代(全 gc) gc 次数  
FGCT：从应用程序启动到采样时 old 代(全 gc ) gc 所用时间(s)  
GCT：从应用程序启动到采样时 gc 用的总时间(s)  

>4. `jstat -gccapacity 进程Id`: 可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小

```bash
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC 
 10240.0 156992.0  10304.0 1024.0 1024.0   8256.0    20480.0   314048.0    20480.0    20480.0      0.0 1077248.0  31360.0      0.0 1048576.0   4224.0     43     1
```

[注]  
NGCMN ：年轻代(young)中初始化(最小)的大小(字节)  
NGCMX ：年轻代(young)的最大容量 (字节)  
NGC ：年轻代(young)中当前的容量 (字节)  
S0C ：年轻代中第一个 survivor（幸存区）的容量 (字节)  
S1C ： 年轻代中第二个 survivor（幸存区）的容量 (字节)  
EC ：年轻代中 Eden（伊甸园）的容量 (字节)  
OGCMN ：old 代中初始化(最小)的大小 (字节)  
OGCMX ：old 代的最大容量(字节)  
OGC：old 代当前新生成的容量 (字节)  
OC ：old 代的容量 (字节)  
MCMN：metaspace (元空间)中初始化(最小)的大小 (字节)  
MCMX ：metaspace (元空间)的最大容量 (字节)  
MC ：metaspace (元空间)当前新生成的容量 (字节)  
CCSMN：最小压缩类空间大小  
CCSMX：最大压缩类空间大小  
CCSC：当前压缩类空间大小  
YGC ：从应用程序启动到采样时年轻代中gc次数  
FGC：从应用程序启动到采样时 old 代(全 gc) gc 次数  



>5. `jstat -gcmetacapacity 进程Id`: metaspace 中对象的信息及其占用量

```bash
MCMN       MCMX        MC       CCSMN      CCSMX       CCSC     YGC   FGC    FGCT     GCT   
   0.0  1077248.0    31360.0        0.0  1048576.0     4224.0    43     1    0.035    0.248
```

[注]  
MCMN：最小元数据容量  
MCMX：最大元数据容量  
MC：当前元数据空间大小  
CCSMN：最小压缩类空间大小  
CCSMX：最大压缩类空间大小  
CCSC：当前压缩类空间大小  
YGC：从应用程序启动到采样时年轻代中gc次数  
FGC：从应用程序启动到采样时 old 代(全 gc) gc 次数  
FGCT：从应用程序启动到采样时 old 代(全gc) gc 所用时间(s)  
GCT：从应用程序启动到采样时 gc 用的总时间(s)  

>6. `jstat -gcnew 进程Id`: 年轻代对象的信息

```bash
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT  
1024.0 1024.0    0.0  538.9  2  15  512.0   8256.0   4125.4     43    0.213
```

[注]  
S0C：年轻代中第一个 survivor（幸存区）的容量 (字节)  
S1C：年轻代中第二个 survivor（幸存区）的容量 (字节)  
S0U：年轻代中第一个 survivor（幸存区）目前已使用空间 (字节)  
S1U：年轻代中第二个 survivor（幸存区）目前已使用空间 (字节)  
TT：持有次数限制  
MTT：最大持有次数限制  
DSS：期望的幸存区大小  
EC：年轻代中 Eden（伊甸园）的容量 (字节)  
EU：年轻代中 Eden（伊甸园）目前已使用空间 (字节)  
YGC：从应用程序启动到采样时年轻代中 gc 次数  
YGCT：从应用程序启动到采样时年轻代中 g c所用时间(s)  


>7. `jstat -gcnewcapacity  进程Id`: 年轻代对象的信息及其占用量

```bash
NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC      YGC   FGC 
10240.0   156992.0    10304.0  15680.0   1024.0  15680.0   1024.0   125632.0     8256.0    43     1
```

[注]  
NGCMN：年轻代(young)中初始化(最小)的大小(字节)  
NGCMX：年轻代(young)的最大容量 (字节)  
NGC：年轻代(young)中当前的容量 (字节)  
S0CMX：年轻代中第一个 survivor（幸存区）的最大容量 (字节)  
S0C：年轻代中第一个 survivor（幸存区）的容量 (字节)  
S1CMX：年轻代中第二个 survivor（幸存区）的最大容量 (字节)  
S1C：年轻代中第二个 survivor（幸存区）的容量 (字节)  
ECMX：年轻代中 Eden（伊甸园）的最大容量 (字节)  
EC：年轻代中Eden（伊甸园）的容量 (字节)  
YGC：从应用程序启动到采样时年轻代中gc次数   
FGC：从应用程序启动到采样时 old 代(全gc)gc次数  


>8. `jstat -gcold  进程Id`: old 代对象的信息

```bash
MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT   
31360.0  29641.5   4224.0   3871.4     20480.0     10523.1     43     1    0.035    0.248
```

[注]  
MC：metaspace(元空间)的容量 (字节)  
MU：metaspace(元空间)目前已使用空间 (字节)  
CCSC：压缩类空间大小  
CCSU：压缩类空间使用大小  
OC：old 代的容量 (字节)  
OU：old 代目前已使用空间 (字节)  
YGC：从应用程序启动到采样时年轻代中 gc次数  
FGC：从应用程序启动到采样时 old 代(全 gc) gc次数  
FGCT：从应用程序启动到采样时 old 代(全gc)gc所用时间(s)  
GCT：从应用程序启动到采样时gc用的总时间(s)  

>9. `jstat -gcoldcapacity  进程Id`: old 代对象的信息及其占用量

```bash
OGCMN       OGCMX        OGC         OC       YGC   FGC    FGCT     GCT   
20480.0    314048.0     20480.0     20480.0    43     1    0.035    0.248
```

[注] 
OGCMN：old 代中初始化(最小)的大小 (字节)
OGCMX：old 代的最大容量(字节)
OGC：old 代当前新生成的容量 (字节)
OC：old 代的容量 (字节)
YGC：从应用程序启动到采样时年轻代中 gc 次数
FGC：从应用程序启动到采样时 old 代(全 gc) gc 次数
FGCT：从应用程序启动到采样时 old 代(全 gc) gc 所用时间(s)
GCT：从应用程序启动到采样时 gc 用的总时间(s)

>10. `jstat -gcutil  进程Id`: 统计gc信息

```bash
S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
0.00  52.62  57.00  51.38  94.52  91.65     43    0.213     1    0.035    0.248
```

[注]  
S0：年轻代中第一个 survivor（幸存区）已使用的占当前容量百分比
S1：年轻代中第二个 survivor（幸存区）已使用的占当前容量百分比
E：年轻代中 Eden（伊甸园）已使用的占当前容量百分比
O：old 代已使用的占当前容量百分比
P：perm 代已使用的占当前容量百分比
YGC：从应用程序启动到采样时年轻代中 gc 次数
YGCT：从应用程序启动到采样时年轻代中 gc 所用时间(s)
FGC：从应用程序启动到采样时 old 代(全 gc) gc 次数
FGCT：从应用程序启动到采样时 old 代(全 gc)gc 所用时间(s)
GCT：从应用程序启动到采样时 gc 用的总时间(s)

>11. `jstat -gccause 进程Id`: 
显示垃圾回收的相关信息（通-gcutil）,同时显示最后一次或当前正在发生的垃圾回收的诱因。

```bash
S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC                 
0.00  52.62  51.97  51.38  94.52  91.65     43    0.213     1    0.035    0.248 Allocation Failure   No GC 
```
[注]  
LGCC：最后一次GC原因  
GCC：当前GC原因（No GC 为当前没有执行GC）  


>12. `jstat -printcompilation  进程Id`: 当前VM执行的信息
```bash
Compiled  Size  Type Method
    2658    209    1 java/util/concurrent/ConcurrentHashMap$Traverser advance
``

[注]  
Compiled：编译任务的数目
Size：方法生成的字节码的大小
Type：编译类型
Method：类名和方法名用来标识编译的方法。类名使用/做为一个命名空间分隔符。方法名是给定类中的方法。上述格式是由-XX:+PrintComplation选项进行设置的 
