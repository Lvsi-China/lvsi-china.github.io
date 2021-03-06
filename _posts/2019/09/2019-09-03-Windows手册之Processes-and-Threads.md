---
layout:     post
title:      【译】Windows手册之进程与线程
subtitle:   Processes , Threads
date:       2019-09-03
author:     Grape
header-img: 
catalog: true
tags:
    - Python
---

# 进程和线程
一个应用程序由一个或多个进程组成。*进程*，用最简单的术语解释，是指一个正在运行的程序，一个或多个线程就运行在某一个进程的上下文中。*线程*，是操作系统分配 *CPU时间(Processor time)* 的基本单位。线程可以执行进程代码的任何部分，包括当前由另一个线程执行的部分。

*作业对象* 允许将一组进程作为一个单元来进行管理，任务对象是可命名的，安全的，可共享的对象，控制与之关联的进程的属性。对作业对象执行的操作会影响与其关联的所有进程。

*线程池* 是由多个worker线程组成的集合，为应用程序提供高效地异步回调。线程池主要用于减少应用程序的线程数并提供对worker线程的管理功能

*纤程* 是必须由应用程序（用户）手动调度的执行单位。纤程运行在调度它们的线程的上下文中。

用户模式调度（UMS）是一种应用程序可以自己调度线程的轻量级机制。UMS线程与纤程的不同之处是每个UMS线程有自己的线程上下文，而不是共享单个线程的线程上下文。

# 关于进程和线程
每个进程都会提供执行程序所需的资源。进程具有虚拟地址空间，可执行代码，系统对象的打开句柄，安全上下文，唯一进程标识符，环境变量，优先级类，最小/最大工作集（Working Set）大小以及至少一个执行线程。每个进程都以一个称为主线程的单个线程启动，但可以从其任何线程中创建其他线程。

线程是在进程中可以被调度执行的一个实体。进程中的所有线程共享其虚拟地址空间和系统资源。此外，每个线程都维护异常处理程序，调度优先级，线程局部存储（Thread Local Storage），唯一线程标识符，以及用于系统保存线程上下文直到被调度的一组结构。线程上下文包括线程的一组机器寄存器，内核栈，线程环境块，和在该线程的进程的地址空间中的用户栈。线程也可以有自己的安全上下文，可用于模拟客户端。

Windows支持*抢占式多任务处理*，它可以实现从多个进程同时执行多个线程的效果。在多处理器计算机上，系统可以同时执行与计算机上的处理器一样多的线程。

线程池主要用于减少应用程序的线程数并提供对worker线程的管理功能。应用程序可以对工作项进行排队，将工作与可等待的句柄相关联，根据计时器自动排队，并与I/O绑定。

用户模式调度（UMS）是一种应用程序可以自己调度线程的轻量级机制。在用户模式下，应用程序可以在UMS线程之间切换，而不涉及系统调度程序，并且如果UMS线程在内核中阻塞，则重新获得对处理器的控制。每个UMS线程都有自己的线程上下文，而不是共享单个线程的线程上下文。在用户模式下，线程之间切换的能力使得UMS比需要很少系统调用的短期工作项的线程池更有效。

*纤程* 是必须由应用程序（用户）手动调度的执行单位。纤程在调度它们的线程的上下文中运行。每个线程可以调度多个纤程。通常，与精心设计的多线程应用相比，纤程并不具有优势。但是，使用纤程可以更轻松地应用于旨在自己调度线程的应用程序上。

## 多任务处理
多任务操作系统将可用的处理器时间划分给需要它的进程或线程上。该系统专为抢占式多任务处理而设计。它为每个执行的线程分配一个处理器时间片。正在执行的线程在其时间片结束时会被挂起，此时另一个线程将会运行。当系统从一个线程切换到另一个线程时，它会保存被抢占了的线程的上下文，并恢复队列中下一个已保存了上下文的线程。

时间片的长度取决于操作系统和处理器。因为每个时间片很小（大约20毫秒），所以多个线程似乎同时在执行。不过，多个线程真正地同时执行，实际上是多处理器系统的情况，其中可执行线程分布在多个可用的处理器上。但是，在应用程序中使用多个线程时必须小心，因为如果线程太多，系统性能会降低。

### 多任务处理的优点
对于用户而言，多任务处理的优势在于能够同时打开和运行多个应用程序。例如，用户可以使用一个应用程序编辑文件，而另一个应用程序正在计算电子表格。

对于应用程序开发人员来说，多任务处理的优势是能够创建使用多个进程的应用程序，并创建使用多个线程的进程。例如，进程可以具有管理与用户的交互的用户界面线程（键盘和鼠标输入），以及在用户界面线程等待用户输入时执行其他任务的工作线程。如果为用户界面线程赋予更高的优先级，则应用程序将对用户更敏感，而工作线程可以在没有用户输入的时间内有效地使用处理器。

### 何时使用多任务处理
有两种实现多任务的方法（单进程多线程和多进程）：作为具有多个线程的单个进程或作为多个进程，每个进程具有一个或多个线程。应用程序可以将需要私有地址空间和私有资源的每个线程放入其自己的进程中，以保护其免受其他进程线程的活动的影响。

由于以下原因，应用程序通过创建单个多线程进程而不是创建多个进程来实现多任务通常更有效：

- 系统可以比线程更快地为线程执行上下文切换，因为进程比线程具有更多的开销（进程上下文大于线程上下文）。
- 进程的所有线程共享相同的地址空间，并且可以访问进程的全局变量，这可以简化线程之间的通信。
- 进程的所有线程都可以共享资源的打开句柄，例如文件和管道。

### 多任务处理注意事项
建议的准则是尽可能少地使用线程，从而最大限度地减少系统资源的使用。这提高了性能。在设计应用程序时，多任务处理需要考虑资源需求和潜在冲突。资源要求如下：
- 系统消耗内存以获取进程和线程所需的上下文信息。因此，可以创建的进程和线程数受可用内存的限制。
- 跟踪大量线程会占用大量处理器时间。如果线程太多，大多数线程将无法取得重大进展。如果大多数当前线程都在一个进程中，则其他进程中的线程调度次数较少。

提供对资源的共享访问可能会产生冲突。要避免它们，您必须同步对共享资源的访问。对于系统资源（例如通信端口），由多个进程（例如文件句柄）共享的资源或由多个线程访问的单个进程（例如全局变量）的资源都是如此。无法正确同步访问（在相同或不同的进程中）可能会导致死锁和竞争条件等问题。您可以使用同步对象和函数来协调多个线程之间的资源共享。减少线程数可以更轻松，更有效地同步资源。

管道服务器是多线程应用程序的一个好设计。在此设计中，您为每个处理器创建一个线程并构建应用程序的队列，应用程序为其维护上下文信息。在处理下一个队列中的请求之前，线程将处理队列中的所有请求。

## 调度

### 调度优先级

### 上下文切换

### 优先级提升

### 优先级倒置

### 多处理器

### 处理器组

## 多线程

### 线程堆栈大小

### 线程句柄和标识符

### 暂停线程执行

### 同步多线程的执行

### 线程本地存储

### 终止线程

### 线程安全和访问权限

## 子进程

### 处理句柄和标识符

### 枚举进程

### 获取其他进程信息

### 终止进程

### 进程安全和访问权限

## 线程池

### Thread Pool API

### Thread Pooling

## 作业对象

## CPU集

## 纤程

## 用户模式调度

