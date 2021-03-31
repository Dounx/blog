---
layout: post
title: 线程中的 fork
categories: linux
---

首先明确在 A 进程的 A1 线程中调用 fork，其中 A 有 A1，A2 以及 A3 线程。那么 fork 出来的进程 B 只会有 A1 的拷贝以及 A 进程的大部分互斥锁，条件变量等，但是不会有 A2 和 A3 线程的拷贝。

fork() 只复制调用 fork 的线程，然后原子线程持有的任何互斥锁将会在 fork 后的子线程中永远锁定。

在 pthread (POSIX Threads) 中避免死锁的方案是 pthread_atfork() 处理程序。

可以注册三个处理程序：prefork，parent 以及 child。

* 在 fork() 启动前调用 prepare 处理程序。
* 在父进程中返回 fork() 后调用 parent 处理程序。
* 在子进程中返回 fork() 后调用 child 处理程序。

例如，prepare 处理程序可能会获取所有需要的互斥锁。然后，parent 和 child 处理程序可能会释放互斥锁。获取所有需要的互斥锁的 prepare 处理程序可确保在对进程执行 fork 之前执行，所有相关的锁定都由调用 fork 函数的线程持有。此技术可防止子进程中出现死锁。

调用 fork() 前会调用 prefork 来获得应用程序所有的互斥锁，父进程和子进程都必须分别释放各自进程中的所有互斥锁。

但是，库调用 pthread_atfork 来注册库特定的互斥锁的处理程序，例如 libc 就会这样做。这是一件好事：应用程序可能无法了解第三方库持有的互斥锁，因此每个库都必须调用 pthread_atfork，以确保在 fork() 事件时清除自己的互斥锁。

问题在于，对不相关的库调用 pthread_atfork 处理程序的顺序是不确定的（这取决于程序加载库的顺序）。因此，这意味着从技术上讲，由于竞态条件，死锁可能会在 fork 之前的处理程序内部发生。

例如：

1. 线程 T1 调用 fork()
2. 在 T1 中调用 libc prefork 处理程序（例如：T1 现在持有所有 libc 的锁）
3. 然后在线程 T2, 一个第三方库 A 获取其自己的互斥锁 AM，然后执行一个需要互斥锁的 libc 调用。这会阻塞, 因为 libc 所有的互斥锁被 T1 持有。
4. T1 调用库 A 的 prefork 处理程序, 这将会阻塞并等待获取被 T2 持有的 AM。

这便是和你的代码无关的死锁（只和你在线程中用的库相关）。
 
最好的方法是进程或者线程二选一，但是对于某些应用来说可能并不是最佳实践。

在多数情况下，如果有一个多进程程序可能会调用 fork()，那么进程里最好不要有多个线程（除了创建进程时自带的主线程）。

## 参考：

1. man 2 fork
2. https://docs.oracle.com/cd/E19253-01/819-7051/gen-74543/index.html
3. https://stackoverflow.com/questions/6078712/is-it-safe-to-fork-from-within-a-thread/6079669#6079669
