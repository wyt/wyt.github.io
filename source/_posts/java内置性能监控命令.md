---
title: java内置性能监控命令
date: 2018-10-5 15:39:00
updated: 2018-10-5 15:39:00
comments: true
---

### jps, JVM Process Status Tool

jps, JVM Process Status Tool,显示指定系统内所有的HotSpot虚拟机进程

``` bash
[root@kvm000 bin]# java -version
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```

<!--more-->

``` bash
[root@kvm000 bin]# jps -l
2368 org.logstash.Logstash
19013 jenkins.war
9210 sun.tools.jps.Jps
2124 org.elasticsearch.bootstrap.Elasticsearch
```

### jstat, JVM Statistics Monitoring Tool

jstat, JVM Statistics Monitoring Tool, 用于收集HotSpot虚拟机各方面的运行数据

``` bash
[root@kvm000 bin]# jstat -gcutil 2124
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
  0.00  26.74  68.99  67.13  92.88  84.44   3661   22.944    14    0.200   23.143
```

E、O、M分别是新生代,老年代,元数据区已用占总大小的百分比
S0、S1表示两个幸存者区
YGC = Young GC, YGCT = Young GC Time 妙计
FGC = Full GC, FGCT = Full GC Time 妙计
GCT, 总的GC 时间.

### jinfo

``` bash
[root@kvm000 bin]# jinfo 2124
Attaching to process ID 2124, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.181-b13
Java System Properties:

jna.platform.library.path = /usr/lib64:/lib64:/usr/lib:/lib:/usr/lib64/mysql
java.runtime.name = Java(TM) SE Runtime Environment
sun.boot.library.path = /usr/local/jdk1.8.0_181/jre/lib/amd64
java.vm.version = 25.181-b13
es.path.home = /opt/elk/elasticsearch-6.2.3
log4j.shutdownHookEnabled = false
java.vm.vendor = Oracle Corporation
java.vendor.url = http://java.oracle.com/
path.separator = :
jna.loaded = true
file.encoding.pkg = sun.io
java.vm.name = Java HotSpot(TM) 64-Bit Server VM
sun.java.launcher = SUN_STANDARD
user.country = US
sun.os.patch.level = unknown
jna.nosys = true
java.vm.specification.name = Java Virtual Machine Specification
user.dir = /opt/elk/elasticsearch-6.2.3
java.runtime.version = 1.8.0_181-b13
java.awt.graphicsenv = sun.awt.X11GraphicsEnvironment
java.endorsed.dirs = /usr/local/jdk1.8.0_181/jre/lib/endorsed
os.arch = amd64
java.io.tmpdir = /tmp/elasticsearch.WqG997rL
line.separator = 

java.vm.specification.vendor = Oracle Corporation
os.name = Linux
io.netty.noKeySetOptimization = true
sun.jnu.encoding = UTF-8
jnidispatch.path = /tmp/elasticsearch.WqG997rL/jna--1985354563/jna2966756306267189304.tmp
java.library.path = /usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
sun.nio.ch.bugLevel = 
es.logs.cluster_name = elasticsearch
java.specification.name = Java Platform API Specification
java.class.version = 52.0
sun.management.compiler = HotSpot 64-Bit Tiered Compilers
os.version = 3.10.0-862.el7.x86_64
user.home = /home/elasticsearch
user.timezone = Asia/Shanghai
java.awt.printerjob = sun.print.PSPrinterJob
file.encoding = UTF-8
java.specification.version = 1.8
io.netty.recycler.maxCapacityPerThread = 0
user.name = elasticsearch
es.logs.base_path = /opt/elk/elasticsearch-6.2.3/logs
java.class.path = /opt/elk/elasticsearch-6.2.3/lib/elasticsearch-6.2.3.jar:/opt/elk/elasticsearch-6.2.3/lib/elasticsearch-core-6.2.3.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-core-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-analyzers-common-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-backward-codecs-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-grouping-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-highlighter-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-join-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-memory-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-misc-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-queries-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-queryparser-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-sandbox-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-spatial-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-spatial-extras-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-spatial3d-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/lucene-suggest-7.2.1.jar:/opt/elk/elasticsearch-6.2.3/lib/securesm-1.2.jar:/opt/elk/elasticsearch-6.2.3/lib/hppc-0.7.1.jar:/opt/elk/elasticsearch-6.2.3/lib/joda-time-2.9.9.jar:/opt/elk/elasticsearch-6.2.3/lib/snakeyaml-1.17.jar:/opt/elk/elasticsearch-6.2.3/lib/jackson-core-2.8.10.jar:/opt/elk/elasticsearch-6.2.3/lib/jackson-dataformat-smile-2.8.10.jar:/opt/elk/elasticsearch-6.2.3/lib/jackson-dataformat-yaml-2.8.10.jar:/opt/elk/elasticsearch-6.2.3/lib/jackson-dataformat-cbor-2.8.10.jar:/opt/elk/elasticsearch-6.2.3/lib/t-digest-3.0.jar:/opt/elk/elasticsearch-6.2.3/lib/HdrHistogram-2.1.9.jar:/opt/elk/elasticsearch-6.2.3/lib/spatial4j-0.6.jar:/opt/elk/elasticsearch-6.2.3/lib/jts-1.13.jar:/opt/elk/elasticsearch-6.2.3/lib/log4j-api-2.9.1.jar:/opt/elk/elasticsearch-6.2.3/lib/log4j-core-2.9.1.jar:/opt/elk/elasticsearch-6.2.3/lib/log4j-1.2-api-2.9.1.jar:/opt/elk/elasticsearch-6.2.3/lib/jna-4.5.1.jar:/opt/elk/elasticsearch-6.2.3/lib/elasticsearch-cli-6.2.3.jar:/opt/elk/elasticsearch-6.2.3/lib/jopt-simple-5.0.2.jar:/opt/elk/elasticsearch-6.2.3/lib/plugin-classloader-6.2.3.jar:/opt/elk/elasticsearch-6.2.3/lib/elasticsearch-launchers-6.2.3.jar:/opt/elk/elasticsearch-6.2.3/lib/plugin-cli-6.2.3.jar
es.path.conf = /opt/elk/elasticsearch-6.2.3/config
java.vm.specification.version = 1.8
sun.arch.data.model = 64
java.home = /usr/local/jdk1.8.0_181/jre
sun.java.command = org.elasticsearch.bootstrap.Elasticsearch -d
user.language = en
java.specification.vendor = Oracle Corporation
io.netty.noUnsafe = true
awt.toolkit = sun.awt.X11.XToolkit
java.vm.info = mixed mode
java.version = 1.8.0_181
java.ext.dirs = /usr/local/jdk1.8.0_181/jre/lib/ext:/usr/java/packages/lib/ext
sun.boot.class.path = /usr/local/jdk1.8.0_181/jre/lib/resources.jar:/usr/local/jdk1.8.0_181/jre/lib/rt.jar:/usr/local/jdk1.8.0_181/jre/lib/sunrsasign.jar:/usr/local/jdk1.8.0_181/jre/lib/jsse.jar:/usr/local/jdk1.8.0_181/jre/lib/jce.jar:/usr/local/jdk1.8.0_181/jre/lib/charsets.jar:/usr/local/jdk1.8.0_181/jre/lib/jfr.jar:/usr/local/jdk1.8.0_181/jre/classes
java.vendor = Oracle Corporation
java.awt.headless = true
file.separator = /
java.vendor.url.bug = http://bugreport.sun.com/bugreport/
sun.io.unicode.encoding = UnicodeLittle
sun.cpu.endian = little
log4j2.disable.jmx = true
sun.cpu.isalist = 

VM Flags:
Non-default VM flags: -XX:+AlwaysPreTouch -XX:CICompilerCount=3 -XX:CMSInitiatingOccupancyFraction=75 -XX:GCLogFileSize=67108864 -XX:+HeapDumpOnOutOfMemoryError -XX:InitialHeapSize=1073741824 -XX:MaxHeapSize=1073741824 -XX:MaxNewSize=348913664 -XX:MaxTenuringThreshold=6 -XX:MinHeapDeltaBytes=196608 -XX:NewSize=348913664 -XX:NumberOfGCLogFiles=32 -XX:OldPLABSize=16 -XX:OldSize=724828160 -XX:-OmitStackTraceInFastThrow -XX:+PrintGC -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintTenuringDistribution -XX:ThreadStackSize=1024 -XX:+UseCMSInitiatingOccupancyOnly -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseConcMarkSweepGC -XX:+UseFastUnorderedTimeStamps -XX:+UseGCLogFileRotation -XX:+UseParNewGC 
Command line:  -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+AlwaysPreTouch -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Djava.io.tmpdir=/tmp/elasticsearch.WqG997rL -XX:+HeapDumpOnOutOfMemoryError -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime -Xloggc:logs/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=32 -XX:GCLogFileSize=64m -Des.path.home=/opt/elk/elasticsearch-6.2.3 -Des.path.conf=/opt/elk/elasticsearch-6.2.3/config
```

### jstat, JVM Statistics Monitoring Tool

``` bash
[root@kvm000 bin]# jstat -gcutil 2124
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
  0.00  26.74  68.99  67.13  92.88  84.44   3661   22.944    14    0.200   23.143
  
  
### jmap

[root@kvm000 bin]# jmap -heap 2124
Attaching to process ID 2124, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.181-b13

using parallel threads in the new generation. #新生代采用的是并行线程处理方式
using thread-local object allocation.
Concurrent Mark-Sweep GC #同步并行垃圾回收

Heap Configuration:
   MinHeapFreeRatio         = 40 #最小堆使用比例
   MaxHeapFreeRatio         = 70 #最大堆可用比例
   MaxHeapSize              = 1073741824 (1024.0MB) #最大堆空间大小
   NewSize                  = 348913664 (332.75MB) #新生代分配大小
   MaxNewSize               = 348913664 (332.75MB) #最大可新生代分配大小
   OldSize                  = 724828160 (691.25MB) #老生代大小
   NewRatio                 = 2 #新生代比例
   SurvivorRatio            = 8 #新生代与suvivor的比例
   MetaspaceSize            = 21807104 (20.796875MB) #元数据区大小
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 314048512 (299.5MB)
   used     = 207005464 (197.4157943725586MB)
   free     = 107043048 (102.0842056274414MB)
   65.91512332973576% used
Eden Space:
   capacity = 279183360 (266.25MB)
   used     = 197681056 (188.52334594726562MB)
   free     = 81502304 (77.72665405273438MB)
   70.80689049662558% used
From Space:
   capacity = 34865152 (33.25MB)
   used     = 9324408 (8.892448425292969MB)
   free     = 25540744 (24.35755157470703MB)
   26.744205790354794% used
To Space:
   capacity = 34865152 (33.25MB)
   used     = 0 (0.0MB)
   free     = 34865152 (33.25MB)
   0.0% used
concurrent mark-sweep generation:
   capacity = 724828160 (691.25MB)
   used     = 486547784 (464.0081253051758MB)
   free     = 238280376 (227.24187469482422MB)
   67.125949411237% used

19955 interned Strings occupying 2899368 bytes.
```

``` bash
# dump jvm
jmap -dump:format=b,file=181005.bin <pid>
```

### jhat, JVM Heap Dump Browser

用于分析heapdump 文件，它会建立一个HTTP/HTML服务器，让用户在浏览器上查看分析结果

### jstack, Stack Trace for Java, 显示虚拟机的线程快找.

https://docs.oracle.com/javase/8/docs/technotes/tools/unix/index.html