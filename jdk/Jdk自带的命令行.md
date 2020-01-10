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


https://www.jianshu.com/u/1f0067e24ff8



