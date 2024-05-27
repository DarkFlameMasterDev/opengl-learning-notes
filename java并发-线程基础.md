---
title: java并发-线程基础（未完成）
date: 2023-04-20 21:16:37
tags: 
- Java
- Jvm
- 线程
- 并发
- 多线程
thumbnail: "https://darkflamemasterdev.github.io/img/language_headimg.png"
---

### 线程的状态

1. New（新建）

   创建后尚未启动。

2. Runnable（运行）

    可能正在运行，也可能正在等待 CPU 时间片。

    包含了操作系统线程状态中的 Running 和 Ready。

3. Blocking（阻塞）

   等待获取一个排它锁，如果其线程释放了锁就会结束此状态。

4. Waiting（无限期等待）

   没有设置 Timeout 参数的 Object.wait() 方法
   没有设置 Timeout 参数的 Thread.join() 方法
   LockSupport.park() 方法

5. Time-Waiting（限期等待）

   进入一个有期限的等待，时间到了自动被唤醒，或者被中断

   > 调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。
   >
   > 调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。
   >
   > 睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。
   >
   > 阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。
   >
   > 而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。

    - Thread.sleep()  
    - 设置了 Timeout 参数的 Object.wait()  
    - 设置了 Timeout 参数的 Thread.join() 方法  
    - LockSupport.parkNanos() 方法，参数等待时间  
    - LockSupport.parkUntil() 方法，参数 deadline  

6. Terminated  
   - 线程自己运行结束
   - 线程因为触发异常而被结束
