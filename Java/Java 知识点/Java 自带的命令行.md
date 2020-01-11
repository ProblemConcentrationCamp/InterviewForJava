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
0x0000000000400000	7K	/opt/jdk-8u162/bin/java
0x00007fa6bc1b9000	91K	/opt/jdk-8u162/jre/lib/amd64/libnio.so
0x00007fa6bcbca000	66K	/usr/lib64/libbz2.so.1.0.6
0x00007fa6bcdda000	153K	/usr/lib64/liblzma.so.5.2.2
0x00007fa6d007d000	88K	/usr/lib64/libz.so.1.2.7

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
class_loader	classes	bytes	parent_loader	alive?	type

<bootstrap>	1640	2901997	  null  	live	<internal>
0x00000000ed186018	1	1471	0x00000000ecdbaeb0	dead	sun/reflect/DelegatingClassLoader@0x0000000100009df8

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






https://www.jianshu.com/u/1f0067e24ff8

https://www.cnblogs.com/kongzhongqijing/articles/3630264.html



