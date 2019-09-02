## 1, 参数

Ø堆设置

Ø`-Xms`
堆初始值，默认物理内存1/64。
-Xms200m

Ø`-Xmx`
堆最大值。
-Xmx1024m

Ø`-Xmn`
年轻代大小

Ø-XX:NewSize

Ø-XX:MaxNewSize

```java
堆设置
-Xms :初始堆大小
-Xmx :最大堆大小
-XX:NewSize=n :设置年轻代大小
-XX:NewRatio=n: 设置年轻代和年老代的比值。如:为3，表示年轻代与年老代比值为1：3，年轻代占整个年轻代年老代和的1/4
-XX:SurvivorRatio=n :年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5
-XX:MaxPermSize=n :设置持久代大小
收集器设置
-XX:+UseSerialGC :设置串行收集器
-XX:+UseParallelGC :设置并行收集器
-XX:+UseParalledlOldGC :设置并行年老代收集器
-XX:+UseConcMarkSweepGC :设置并发收集器
垃圾回收统计信息
-XX:+PrintGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:filename
并行收集器设置
-XX:ParallelGCThreads=n :设置并行收集器收集时使用的CPU数。并行收集线程数。
-XX:MaxGCPauseMillis=n :设置并行收集最大暂停时间
-XX:GCTimeRatio=n :设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n)
并发收集器设置
-XX:+CMSIncrementalMode :设置为增量模式。适用于单CPU情况。
-XX:ParallelGCThreads=n :设置并发收集器年轻代收集方式为并行收集时，使用的CPU数。并行收集线程数。

```



## 2, 帮助文档中的内容

```shell
$ java -X
    -Xmixed           mixed mode execution (default)
    -Xint             interpreted mode execution only
    -Xbootclasspath:<directories and zip/jar files separated by :>
                      set search path for bootstrap classes and resources
    -Xbootclasspath/a:<directories and zip/jar files separated by :>
                      append to end of bootstrap class path
    -Xbootclasspath/p:<directories and zip/jar files separated by :>
                      prepend in front of bootstrap class path
    -Xdiag            show additional diagnostic messages
    -Xnoclassgc       disable class garbage collection
    -Xincgc           enable incremental garbage collection
    -Xloggc:<file>    log GC status to a file with time stamps
    -Xbatch           disable background compilation
    
    -Xms<size>        set initial Java heap size
    									java堆内存的初始值大小
    									
    -Xmx<size>        set maximum Java heap size
    									java堆内存的最大值大小
    									
    -Xss<size>        set java thread stack size
    									java栈区的大小，默认好像是1m
    
    -Xprof            output cpu profiling data
    -Xfuture          enable strictest checks, anticipating future default
    -Xrs              reduce use of OS signals by Java/VM (see documentation)
    -Xcheck:jni       perform additional checks for JNI functions
    -Xshare:off       do not attempt to use shared class data
    -Xshare:auto      use shared class data if possible (default)
    -Xshare:on        require using shared class data, otherwise fail.
    -XshowSettings    show all settings and continue
    -XshowSettings:all
                      show all settings and continue
    -XshowSettings:vm show all vm related settings and continue
    -XshowSettings:properties
                      show all property settings and continue
    -XshowSettings:locale
                      show all locale related settings and continue

The -X options are non-standard and subject to change without notice.


The following options are Mac OS X specific:
    -XstartOnFirstThread
                      run the main() method on the first (AppKit) thread
    -Xdock:name=<application name>"
                      override default application name displayed in dock
    -Xdock:icon=<path to icon file>
                      override default icon displayed in dock
```

