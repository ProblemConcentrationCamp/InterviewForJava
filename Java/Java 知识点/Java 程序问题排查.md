# Java 程序问题排查

## 定位 Java 程序高 CPU
>1. 先找到高 CPU 的程序 -> `top` -> 定位到高 CPU 的 Java 的进程 Id: A
>2. 找到高 CPU 程序内的高 CPU 的线程 -> `top -H -p A(第一步找到的进行 Id)` -> 线程 Id: B
>3. 将找到的线程 Id 转为 16 进制 -> `printf "%x\n" B(第二步得到的线程Id)` -> 16进制: C
>4. 通过 jstack 找到对应的线程 -> `jstack A | grep c -A 30`
>5. 分析线程信息