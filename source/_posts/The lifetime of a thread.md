---
title: The lifetime of a thread
date: 2019-8-22 19:40:00
updated: 2019-8-22 19:40:00
comments: true
categories:
- thread
tags:
- thread
- java
---

# The lifetime of a thread

### 线程的状态

线程有六种状态，定义在`java.lang.Thread.State`内部枚举中：

```java
public enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;
}
```

<!-- more -->

A thread state. A thread can be in one of the following states:

- **NEW** A thread that has not yet started is in this state.
- **RUNNABLE** A thread executing in the Java virtual machine is in this state.
- **BLOCKED** A thread that is blocked waiting for a monitor lock is in this state. 处于此状态，线程被阻塞等待监视器锁
- **WAITING** A thread that is waiting indefinitely for another thread to perform a particular action is in this state. 无限期等待另外线程执行特定的动作，通常为唤醒等操作
- **TIMED_WAITING** A thread that is waiting for another thread to perform an action for up to a specified waiting time is in this state. 限期等待
- **TERMINATED** A thread that has exited is in this state.

A thread can be in only one state at a given point in time. These states are virtual machine states which do not reflect any operating system thread states.

### 线程的生命周期
![The lifetime of a thread.gif](https://objects.yongtao.wang/images/20190903/java-thread-life-cycle-state.gif)
实现三个线程，其中Thread-01执行睡眠，Thread-02、Thread-03执行同步任务，jstack抓取堆栈信息，观察线程状态。
```java
public static void main(String[] args) {
    Thread thd01 =
        new Thread(
        () -> {
            try {
                TimeUnit.SECONDS.sleep(120);
            } catch (InterruptedException e) {
                // ignore
            }
        },
        "Thread-01");

    Thread thd02 =
        new Thread(
        () -> {
            doSorting();
        },
        "Thread-02");

    Thread thd03 =
        new Thread(
        () -> {
            doSorting();
        },
        "Thread-03");

    thd01.start();
    thd02.start();
    thd03.start();
}

public static synchronized void doSorting() {
    for (; ; ) {
        // 生成随机数组
        double[] a = new double[100];
        for (int i = 0; i < a.length; i++) {
            a[i] = new Random().nextDouble();
        }
        // 对随机数组进行排序，使其消耗CPU
        Arrays.sort(a);
    }
}
```
如堆栈信息所示，Thread-01处于TIMED_WAITING状态，Thread-02处于RUNNABLE状态，Thread-02处于BLOCKED状态。
```
"Thread-03" #13 prio=5 os_prio=0 tid=0x000000002092b000 nid=0x345c waiting for monitor entry [0x00000000214fe000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at io.mysnippet.multithread.ThreadState.doSorting(ThreadState.java:49)
        - waiting to lock <0x000000068280a830> (a java.lang.Class for io.mysnippet.multithread.ThreadState)
        at io.mysnippet.multithread.ThreadState.lambda$main$2(ThreadState.java:37)
        at io.mysnippet.multithread.ThreadState$$Lambda$3/1078694789.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None

"Thread-02" #12 prio=5 os_prio=0 tid=0x000000002092a000 nid=0x748 runnable [0x00000000213fe000]
   java.lang.Thread.State: RUNNABLE
        at io.mysnippet.multithread.ThreadState.doSorting(ThreadState.java:50)
        - locked <0x000000068280a830> (a java.lang.Class for io.mysnippet.multithread.ThreadState)
        at io.mysnippet.multithread.ThreadState.lambda$main$1(ThreadState.java:30)
        at io.mysnippet.multithread.ThreadState$$Lambda$2/1096979270.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None

"Thread-01" #11 prio=5 os_prio=0 tid=0x0000000020949800 nid=0x4494 waiting on condition [0x00000000212ff000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at java.lang.Thread.sleep(Thread.java:340)
        at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
        at io.mysnippet.multithread.ThreadState.lambda$main$0(ThreadState.java:20)
        at io.mysnippet.multithread.ThreadState$$Lambda$1/2003749087.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None
```