---
layout: post
title:  " Android开篇"
permalink: /android/
date:   2016-04-01 11:49:40
catalog:  true
tags:
    - android


---

> **版权声明：**本站所有博文内容均为原创，欢迎交流与转载，转载请务必注明作者与原文链接，且不得轻易篡改原文内容。另外，未经授权不得用于任何商业目的。


## 一、简述

Android系统非常庞大、错中复杂，其底层是采用Linux作为基底，上层采用包含虚拟机的Java层以及Native层，通过系统调用(Syscall)连通系统的内核空间与用户空间。用户空间主要采用C++和Java代码，通过JNI技术打通用户空间的Java层和Native层(C++/C)，从而融为一体。

Google官方提供了一张[经典的四层架构图](http://gityuan.com/2016/01/30/android-boot/#android)，从下往上依次分为Linux内核、系统库和Android运行时环境、框架层以及应用层这4层架构，其中每一层都包含大量的子模块或子系统。这只是如垒砖般地简单分层，还远不足以表达Android整个系统的内部架构，运行机理，以及各个模块之间是如何衔接与配合工作的。为了更深入地掌握Android整个架构思想以及各个模块在Android系统所处的地位与价值，计划以Android系统启动过程为主线，**以进程的视角**来诠释Android系统全貌，全方位的深度剖析各个模块功能，争取各个击破。

**本站所有Android系列的文章，都是基于Android 6.0源码**，深入剖析Android系统架构，力争各个击破。这样才能犹如庖丁解牛，解决、分析问题则能游刃有余。

## 二、Android架构

**系统启动架构图**

点击查看[大图](http://gityuan.com/images/android-process/android-boot.jpg)

![process_status](/images/android-process/android-boot.jpg)
  
  
**图解：**  
Android系统启动过程由上图从下往上的一个过程：`Loader` -> `Kernel` -> `Native` -> `Framework` -> `App`，接来下简要说说每个过程：

### 2.1 Loader层

- Boot ROM: 当手机处于关机状态时，长按Power键开机，引导芯片开始从固化在`ROM`里的预设出代码开始执行，然后加载引导程序到`RAM`；
- Boot Loader：这是启动Android系统之前的引导程序，主要是检查RAM，初始化硬件参数等功能。
	
### 2.2 Kernel层

Kernel层是指Android内核层，到这里才刚刚开始进入Android系统。

- 启动Kernel的0号进程：初始化进程管理、内存管理，加载Display,Camera Driver，Binder Driver等相关工作；
- 启动kthreadd进程（pid=2）：是Linux系统的内核进程，会创建内核工作线程kworkder，软中断线程ksoftirqd，thermal等内核守护进程。`kthreadd进程是所有内核进程的鼻祖`。
	
### 2.3 Native层

这里的Native层主要包括init孵化来的用户空间的守护进程、HAL层以及开机动画等。启动init进程(pid=1),是Linux系统的用户进程，`init进程是所有用户进程的鼻祖`。

- init进程会孵化出ueventd、logd、healthd、installd、adbd、lmkd等用户守护进程；
- init进程还启动`servicemanager`(binder服务管家)、`bootanim`(开机动画)等重要服务
- init进程孵化出Zygote进程，Zygote进程是Android系统的第一个Java进程，`Zygote是所有Java进程的父进程`，Zygote进程本身是由init进程孵化而来的。

### 2.4 Framework层

- Zygote进程，是由init进程通过解析init.rc文件后fork生成的，Zygote进程主要包含：
	- 加载ZygoteInit类，注册Zygote Socket服务端套接字；
	- 加载虚拟机；
	- preloadClasses；
	- preloadResouces。
- System Server进程，是由Zygote进程fork而来，`System Server是Zygote孵化的第一个进程`，System Server负责启动和管理整个Java framework，包含ActivityManager，PowerManager等服务。
- Media Server进程，是由init进程fork而来，负责启动和管理整个C++ framework，包含AudioFlinger，Camera Service，等服务。
	
### 2.5 App层

- Zygote进程孵化出的第一个App进程是Launcher，这是用户看到的桌面App；
- Zygote进程还会创建Browser，Phone，Email等App进程，每个App至少运行在一个进程上。
- 所有的App进程都是由Zygote进程fork生成的。

### 2.6 Syscall && JNI

- Native与Kernel之间有一层系统调用(SysCall)层，见[Linux系统调用(Syscall)原理](http://gityuan.com/2016/05/21/syscall/);
- Java层与Native(C/C++)层之间的纽带JNI，见[Android JNI原理分析](http://gityuan.com/2016/05/28/android-jni/)。

##  三、通信方式

无论是Android系统，还是各种Linux衍生系统，各个组件、模块往往运行在各种不同的进程和线程内，这里就必然涉及进程/线程之间的通信。对于IPC(Inter-Process Communication, 进程间通信)，Linux现有管道、消息队列、共享内存、套接字、信号量、信号这些IPC机制，Android额外还有Binder IPC机制，Android OS中的Zygote进程的IPC采用的是Socket机制，在上层system server、media server以及上层App之间更多的是采用Binder IPC方式来完成跨进程间的通信。对于Android上层架构中，还多时候是在同一个进程的线程之间需要相互通信，例如同一个进程的主线程与工作线程之间的通信，往往采用的Handler消息机制。

想深入理解Android内核层架构，必须先深入理解Linux现有的IPC机制；对于Android上层架构，则最常用的通信方式是Binder、Socket、Handler，当然也有少量其他的IPC方式，比如杀进程Process.killProcess()采用的是signal方式。下面说说Binder、Socket、Handler：


### 3.1 Binder 

Binder作为Android系统提供的一种IPC机制，无论从系统开发还是应用开发，都是Android系统中最重要的组成，也是最难理解的一块知识点，想了解[为什么Android要采用Binder作为IPC机制？](https://www.zhihu.com/question/39440766/answer/89210950)，可查看博主在知乎上的回答。深入了解Binder机制，最好的方法便是阅读源码，借用Linux鼻祖Linus Torvalds曾说过的一句话：Read The Fucking Source Code。下面简要说说Binder IPC原理。

**Binder IPC原理**

Binder通信采用c/s架构，从组件视角来说，包含Client、Server、ServiceManager以及binder驱动，其中ServiceManager用于管理系统中的各种服务。

![ServiceManager](/images/binder/prepare/IPC-Binder.jpg)
	
- 想进一步了解Binder，可查看[Binder系列—开篇](http://gityuan.com/2015/10/31/binder-prepare/)，Binder系列用了13篇文章，从源码角度出发来，讲述驱动、native、framework、app四个层面的整个完整流程。

### 3.2 Socket

Socket通信方式也是C/S架构，比Binder简单很多。在Android系统中采用Socket通信方式的主要：

- zygote：用于孵化进程，系统进程system_server孵化进程时便通过socket向zygote进程发起请求；
- installd：用于安装App的守护进程，上层PackageManagerService很多实现最终都是交给它来完成；
- lmkd：lowmemorykiller的守护进程，Java层的LowMemoryKiller最终都是由lmkd来完成；
- adbd：这个也不用说，用于服务adb；
- logcatd:这个不用说，用于服务logcat；
- vold：即volume Daemon，是存储类的守护进程，用于负责如USB、Sdcard等存储设备的事件处理。

等等还有很多，这里不一一列举，Socket方式更多的用于Android framework层与native层之间的通信。Socket通信方式相对于binder非常简单，所以一直没有写相关文章，为了成一个体系，下次再补上。

### 3.3 Handler

**Binder/Socket用于进程间通信，而Handler消息机制用于同进程的线程间通信**，Handler消息机制是由一组MessageQueue、Message、Looper、Handler共同组成的，为了方便且称之为Handler消息机制。

有人可能会疑惑，为何Binder/Socket用于进程间通信，能否用于线程间通信呢？答案是肯定，对于两个具有独立地址空间的进程通信都可以，当然也能用于共享内存空间的两个线程间通信，这就好比杀鸡用牛刀。接着可能还有人会疑惑，那handler消息机制能否用于进程间通信？答案是不能，Handler只能用于共享内存地址空间的两个线程间通信，即同进程的两个线程间通信。很多时候，Handler是工作线程向UI主线程发送消息，即App应用中只有主线程能更新UI，其他工作线程往往是完成相应工作后，通过Handler告知主线程需要做出相应地UI更新操作，Handler分发相应的消息给UI主线程去完成，如下图：

![handler_communication](/images/handler/handler_thread_commun.jpg)

由于工作线程与主线程共享地址空间，即Handler实例对象`mHandler`位于线程间共享的内存堆上，工作线程与主线程都能直接使用该对象，只需要注意多线程的同步问题。工作线程通过`mHandler`向其成员变量`MessageQueue`中添加新Message，主线程一直处于loop()方法内，当收到新的Message时按照一定规则分发给相应的`handleMessage`()方法来处理。所以说，而Handler消息机制用于同进程的线程间通信的核心是线程间共享内存空间，而不同进程拥有不同的地址空间，也就不能用handler来实现进程间通信。

上图只是Handler消息机制的一种处理流程，是不是只能工作线程向UI主线程发消息呢，其实不然，可以是UI线程想工作线程发送消息，也可以是多个工作线程之间通过handler发送消息。更多关于Handler消息机制文章：

- [Android消息机制-Handler(framework篇)](http://gityuan.com/2015/12/26/handler-message-framework/)
- [Android消息机制-Handler(native篇)](http://gityuan.com/2015/12/27/handler-message-native/)

##  四、计划提纲

2016年新的一年刚开始，首先祝大家、也祝自己在新的一年诸事顺心，事业蒸蒸日上。在过去的一年，对于Android从底层一路到上层有不少自己的理解和沉淀，但总体较零散，未成体系。借着今天（元旦假日的最后一天），给自己的新的一年提前做一个计划，把知识进行归档整理与再学习，从而加深对Android架构的理解。通过前面对系统启动的介绍，相信大家对Android系统有了一个整体观，接下来需要抓核心、理思路。


**（1）**Android系统启动过程中，有几个非常重要的进程：`init`、`Zygote`、`system_server`进程:

- [Android系统启动—init篇](http://gityuan.com/2016/02/05/android-init/)
- [Android系统启动—Zygote篇](http://gityuan.com/2016/02/13/android-zygote/)
- Android系统启动—SystemServer篇
	- [SystemServer上篇](http://gityuan.com/2016/02/14/android-system-server/)
	- [SystemServer下篇](http://gityuan.com/2016/02/20/android-system-server-2/)
  
**（2）**再则就是在整个架构中有大量的服务，都是基于[Binder](http://gityuan.com/2015/10/31/binder-prepare/)来交互的，计划针对部分核心服务来重点分析：

- Android服务篇-ActivityManagerService
	- [AMS启动过程（一）](http://gityuan.com/2016/02/21/activity-manager-service/)
- Android服务篇-PackageManagerService
- Android服务篇-PowerManagerService
- Android服务篇-BatteryService
	- [Android耗电统计算法](http://gityuan.com/2016/01/10/power_rank/)
- Android服务篇-WindowManagerService
  
当然graphic也是一大块难啃的模块，也是需要整理的，先留个空位吧。

**（3）**对于App来说，Android应用的四大组件Activity，Service，Broadcast Receiver， Content Provider最为核心，那么我们需要分别展开对其他的分解：

- Android组件-Activity
	- [startActivity流程分析(一)](http://gityuan.com/2016/03/12/start-activity/)
- Android组件-Service
	- [startService流程分析](http://gityuan.com/2016/03/06/start-service/)
- Android组件-Broadcast Receiver
- Android组件-Content Provider

  
**（4）**有了这些，中间还缺少关于虚拟机ART的介绍，会需要对ART分析，后续还需要开展对ART虚拟机的一系列文章。回顾整个架构，谈谈系统性能，需要先掌握进程、内存、IO这些层面知识，这里牵涉面较广，从底层Linux层直至上层App

- 进程篇
	- [理解Android进程创建流程](http://gityuan.com/2016/03/26/app-process-create/)
	- [理解杀进程的实现原理](http://gityuan.com/2016/04/16/kill-signal/)
	- [Android进程整理](http://gityuan.com/2015/12/19/android-process-category/)
	- [Android进程生命周期与ADJ](http://gityuan.com/2015/10/01/process-lifecycle/)
	- [进程优先级](http://gityuan.com/2015/10/01/process-priority/)
- 内存篇
	- [Linux内存管理](http://gityuan.com/2015/10/30/kernel-memory/)
- IO篇
- Linux驱动篇
 
**（5）**最后，说说Android相关的一些常用命令和工具

- [理解Android编译命令](http://gityuan.com/2016/03/19/android-build/)
- [性能工具Systrace](http://gityuan.com/2016/01/17/systrace/)
- [Android内存分析命令](http://gityuan.com/2016/01/02/memory-analysis-command/)
- [ps进程命令](http://gityuan.com/2015/10/11/ps-command/)
- [Am命令用法](http://gityuan.com/2016/02/27/am-command/)
- [Pm命令用法](http://gityuan.com/2016/02/28/pm-command/)
- [dumpsys命令用法](http://gityuan.com/2016/05/14/dumpsys-command/)
  
  
本博客还有很多文章并没有写到上面这个清单，先写这么多，后续再不断更新与完善。最后，欢迎大家交流与纠错，大家来找茬。