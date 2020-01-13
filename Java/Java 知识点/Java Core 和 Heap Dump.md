# Java Core 和 Heap Dump

当 Java 程序在遇到致命问题导致程序结束时，JVM会在程序停止前产生两个文件，分别为 JavaCore 及 HeapDump 文件。

|  | Java Core  | Heap Dump |
| :-: | :-: | :-: |
| 文件类型 | 文本 | 二进制文件 |
| 文件内容 | 各线程在程序某一个时刻的线程堆栈 | 程序在某一个时刻, 整个 JVM 堆的对象分布信息 |
| 作用 | 分析进程挂死, 响应速度慢, 大对象 | 分析内存泄漏 大对象|
|分析工具 | IBM Thread and Monitor Dump Analyzer for Java | IBM HeapAnalyzer|
| 影响 | 对系统影响小,生成文件小,比较方便 | 对系统影响大,生成文件大,较少使用|



## Java Core 的生成方式

1. oracle JDK 的 `java_home/bin/jvisualvm.exe` 可以生产当前程序的 Java Core。
打开这个执行程序，然后在应用程序选择你的程序，然后在右边顶部的 Tag 栏，选择里面的 
线程，在里面可以看到一个 `线程 Dump` 的按钮。点击他就能获取到当前程序的 Java Core 信息。
2. 通过 jstack 命令生成线程堆栈信息，同时将信息导入到一个新的文件 `jstack -l [进程Id] >> 文件名.文件后缀`
3. `kill -3 进程号` 向应用程序发送一个中断信息
4. 不需要任何配置, 在项目崩溃的时候，会自动在程序所在的目录生成 Java Core 文件，hs_err_pid<pid>.log是java程序发生core的时候产生的文件。（此条待验证）


## Heap Dump 的生成方式
1. oracle JDK 的 `java_home/bin/jvisualvm.exe` 可以生产当前程序的 Heap Dump。打开这个执行程序，然后在应用程序选择你的程序，然后在右边顶部的 Tag 栏，选择里面的监视，在里面可以看到一个 `堆 Dump` 的按钮。点击他就能获取到当前程序的 Heap Dump 信息。
2. 通过 jmap 命令生成 Heap Dump, `jmap -dump:format=b,file=heap.dump(文件名) 进程Id`
3. oracle jdk 可以配置在程序崩溃的时候，生成 heap dump 在启动命令行参数里面加上 `-XX:+HeapDumpOnOutOfMemoryError`