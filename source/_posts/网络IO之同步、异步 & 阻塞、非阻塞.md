---
title: 网络IO之同步、异步 & 阻塞、非阻塞
date: 2019-3-8 14:41:27
updated: 2019-3-8 14:41:33
comments: true
tags:
- OS
---

同步(Synchronous)

A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes; (同步I/O操作导致请求进程被阻塞，直到I/O操作完成)

异步(Asynchronous)

An asynchronous I/O operation does not cause the requesting process to be blocked; (异步I/O操作不会阻塞请求进程)

注意Non-blocking IO在执行recvfrom这个系统调用的时候，如果kernel的数据没有准备好，不会block进程；如果kernel数据准备好了，recvfrom会将数据从内核空间拷贝到用户空间，在这段时间内，进程还是被block的。

阻塞(Blocking)

阻塞调用是指调用结果返回之前，当前线程会被挂起。

非阻塞(Non-blocking)

非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。

http://www.rowkey.me/blog/2016/01/18/io-model/
https://baiweiblog.wordpress.com/tag/non-blocking/
https://www.zhihu.com/question/19732473/answer/20851256
https://blog.csdn.net/historyasamirror/article/details/5778378