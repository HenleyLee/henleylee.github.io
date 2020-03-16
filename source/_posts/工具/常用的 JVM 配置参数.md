---
title: 常用的 JVM 配置参数
categories: 工具
tags:
  - IntelliJ IDEA
abbrlink: 399ccee7
date: 2019-08-21 15:38:32
---

1. **`-Xms：`**初始堆大小。只要启动，就占用的堆大小。
2. **`-Xmx：`**最大堆大小。java.lang.OutOfMemoryError：Java heap 这个错误可以通过配置 -Xms 和 -Xmx 参数来设置。
3. **`-Xss：`**栈大小分配。栈是每个线程私有的区域，通常只有几百K大小，决定了函数调用的深度，而局部变量、参数都分配到栈上。
当出现大量局部变量，递归时，会发生栈空间 OOM（java.lang.StackOverflowError）之类的错误。
4. **`-XX:NewSize：`**设置新生代大小的绝对值。
5. **`-XX:NewRatio：`**设置年轻代和年老代的比值。比如设置为3，则新生代：老年代=1:3，新生代占总 heap 的1/4。
6. **`-XX:MaxPermSize：`**设置持久代大小。
java.lang.OutOfMemoryError:PermGenspace 这个 OOM 错误需要合理调大 PermSize 和 MaxPermSize 大小。
7. **`-XX:SurvivorRatio：`**年轻代中 Eden 区与两个 Survivor 区的比值。注意，Survivor 区有 form 和 to 两个。比如设置为8时，那么 eden:form:to=8:1:1。
8. **`-XX:HeapDumpOnOutOfMemoryError：`**发生 OOM 时转储堆到文件，这是一个非常好的诊断方法。
9. **`-XX:HeapDumpPath：`**导出堆的转储文件路径。
10. **`-XX:OnOutOfMemoryError：`**OOM 时，执行一个脚本，比如发送邮件报警，重启程序。后面跟着一个脚本的路径。

