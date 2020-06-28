---
title: QNX 编程
date: 2013-07-28T08:00:00+08:00
draft: false
toc:
comments: true
---


处理中断的线程应该先获得 I/O 权限，调用 ThreadCtl() 函数：

	#include <sys/neutrino.h>
	ThreadCtl( _NTO_TCTL_IO, 0 );

QNX 提供了两种捆绑中断服务程序和中断号的方式：`InterruptAttach()` 和 `InterruptAttachEvent()` 。两种方式调用中断服务程序的方式也不同。分离可以调用 `InterruptDetach()` 函数：

	#define IRQ3 3
	
	/*  A forward reference for the handler */
	extern const sigevent *serint (void *, int);
	…
	
	/*
	 *  Associate the interrupt handler, serint,
	 *  with IRQ 3, the 2nd PC serial port
	 */
	ThreadCtl( _NTO_TCTL_IO, 0 );
	id = InterruptAttach (IRQ3, serint, NULL, 0, 0);
	…
	
	/*  Perform some processing. */
	…
	
	/*  Done; detach the interrupt source. */
	InterruptDetach (id);

## 中断服务程序

中断服务程序中应该完成如下几个工作：

1. 判断中断源

	根据你的硬件配置，可能出现多个硬件共享一个中断号。大部分PIC(可编程中断控制器)芯片可以编程配置为电平触发中断或边沿触发中断。触发方式决定了中断是否可以共享。

	下面是多中断源请求中断的示意图：

    ![](~/IMG_2491.JPG)

	在上面这种情况下，如果 PIC 配置为电平触发，只要电平为高，就认为 IRQ 处于激活状态。第二个中断请求(step2)本身并不会产生新的中断，中断一直被认为处于激活状态，即使最初引起中断的中断源已经删除(step3)。直到最后一个中断源被清除，该中断才会被认为处于未激活状态。

	如果是边沿触发模式，中断只会被“注意”一次，即 step1 。只有当中断线被清除，然后被再次请求，PIC 才会认为有新的中断产生。

	QNX 允许多个中断服务程序关联一个中断号。这样的话，每个服务程序必须找到自己关联的硬件并且判断是否是由该硬件引起的中断。这项工作在电平触发环境下是可靠的，边沿触发环境就不一定了。

2. 处理硬件

	通常是读写硬件寄存器。

	需要注意的是，在 ISR 中不能使用大部分内核调用。在使用库函数时也要小心，因为它们的底层实现可能使用了系统调用，例如 `printf()` 函数。

	在 ISR 中可以使用的内核调用有：

		* InterruptMask()
		* InterruptUnmask()
		* TraceEvent()

	`InterruptMask()` 和 `InterruptUnmask()` 用于关闭和使能一个特定的中断号。
	`TraceEvent()` 用于跟踪内核事件。

3. 更新数据结构

	另一个问题是如何安全的更新 ISR 和软件线程共同使用的数据结构。有两个特点值得重申：

	* ISR 拥有比任何软件线程都高的优先级
	* ISR 不能使用内核调用(特别声明的除外)

	这意味着不能在 ISR 中使用线程级别的同步(例如互斥)。

4. 唤醒应用层代码

	因为 ISR 中的操作受到很大限制，通常需要将大量的操作放在线程级别运行

	为了做到这一点，有两个选择：

	* 在 ISR 中完成一些对时限要求严格的工作，其他的安排给唤醒的线程来处理
	* 在 ISR 中什么都不做，只是唤醒一个线程

	这也是 `InterruptAttach()` 和 `InterruptAttachEvent()` 的区别。


## 使用 InterruptAttach

`InterruptAttach()` 函数的原型：

	#include <sys/neutrino.h>
	
	int InterruptAttach( int intr,
	       const struct sigevent * (* handler)(void *, int),
	       const void * area,
	       int size,
	       unsigned flags );

参数：

* intr : 中断号
* handler : 中断服务函数
* area : 线程与中断服务函数进行通讯的空间，如果不需要可以设为NULL
* size : 通讯空间的大小
* flags : 表示捆绑中断服务函数的方式，有三个可选值：

	* \_NTO\_INTR\_FLAGS\_END  : 将新的中断服务函数放在该中断的处理函数队列的尾部。  

	* \_NTO\_INTR\_FLAGS\_PROCESS  : 将中断服务函数关联到进程，而不是当前线程.

	* \_NTO\_INTR\_FLAGS\_TRK\_MSK  : 跟踪调用 `InterruptMask()` 和 `InterruptUnmask()` 来安全的分离中断服务函数。 

调用成功的话会返回一个 id ，用于 `InterruptMask()` 和 `InterruptUnmask()` 等函数。失败返回-1。

中断服务函数(ISR)的原型是：

	const struct sigevent *handler (void *area, int id);

参数：

* area : 即 `InterruptAttach()` 函数的 `area` 
* id : 即 `InterruptAttach()` 函数的返回值

在 `handler()` 函数中完成了相应的中断服务后，可以选择安排一个线程来完成后续的工作。为此，ISR 需要返回一个指向 `const struct sigevent` 结构的指针，当内核收到这个结构后，就会将 event 带到目的地。

`struct sigevent` 定义在 `<sys/siginfo.h>` 头文件，结构成员 `sigev_notify` 指定了产生怎样的通知。该文件提供了一些宏来初始化这个结构，第一个参数都是 `struct sigevent` 类型的指针，中断服务函数中常用的宏有如下两个：

* SIGEV\_INTR\_INIT(&event)

	设置 `sigev_notify` 为 `SIGEV_INTR`，表示发送一个中断通知到指定的线程。

* SIGEV\_NONE\_INIT(&event)

	设置 `sigev_notify` 为 `SIGEV_NONE`，表示不发送任何通知。

典型的处理中断的方法是，在一个线程中调用 `InterruptAttach()` 绑定中断服务函数，然后调用 `InterruptWait()` 阻塞线程，直到 ISR 返回一个 `SIGEV_INTR` ，`InterruptWait()` 才会返回：

	main ()
	{
	    // perform initializations, etc.
	    …
	    // start up a thread that is dedicated to interrupt processing
	    pthread_create (NULL, NULL, int_thread, NULL);
	    …
	    // perform other processing, as appropriate
	    …
	}
	
	// this thread is dedicated to handling and managing interrupts
	void *
	int_thread (void *arg)
	{
	    // enable I/O privilege
	    ThreadCtl (_NTO_TCTL_IO, NULL);
	    …
	    // initialize the hardware, etc.
	    …
	    // attach the ISR to IRQ 3
	    InterruptAttach (IRQ3, isr_handler, NULL, 0, 0);
	    …
	    // perhaps boost this thread's priority here
	    …
	    // now service the hardware when the ISR says to
	    while (1)
	    {
	        InterruptWait (NULL, NULL);
	        // at this point, when InterruptWait unblocks,
	        // the ISR has returned a SIGEV_INTR, indicating
	        // that some form of work needs to be done.
	
	
	        …
	        // do the work
	        …
	        // if the isr_handler did an InterruptMask, then
	        // this thread should do an InterruptUnmask to
	        // allow interrupts from the hardware
	    }
	}
	
	// this is the ISR
	const struct sigevent *
	isr_handler (void *arg, int id)
	{
	    // look at the hardware to see if it caused the interrupt
	    // if not, simply return (NULL);
	    …
	    // in a level-sensitive environment, clear the cause of
	    // the interrupt, or at least issue InterruptMask to
	    // disable the PIC from reinterrupting the kernel
	    …
	    // return a pointer to an event structure (preinitialized
	    // by main) that contains SIGEV_INTR as its notification type.
	    // This causes the InterruptWait in "int_thread" to unblock.
	    return (&event);
	}

这种方案相对于使用 `SIGEV_SIGNAL` 或 `SIGEV_PULSE` 类型有几个优点：

* 应用程序无需调用 `MsgReceive()` 函数(用于等到一个pulse)。
* 应用程序无需一个 signal-handler 函数(用于等待一个signal)。
* 如果中断处理很急迫，可以定义一个具有高优先级的 `int_thread()` 线程；当 `isr_handler()` 返回 SIGEV_INTR 时，只要 `int_thread()` 的优先级足够高，它就会立即执行。

使用 `InterruptWait()` 时需要注意一点，被绑定的中断必须产生 `SIGEV_INTR` 。

## 使用 InterruptAttachEvent

`InterruptAttachEvent()` 函数的原型：

	int InterruptAttachEvent( 
	       int intr,
	       const struct sigevent* event,
	       unsigned flags );

前面关于 `InterruptAttach()` 的讨论大部分都适用于 `InterruptAttachEvent()` ，除了 ISR 。这里不需要提供 ISR ,`InterruptAttachEvent()` 会自己处理中断，因为已经为中断号绑定了一个 `struct sigevetn` 结构，内核可以检查到这个事件。这样的好处是我们无需考虑进入 ISR 以及如何返回。

需要注意的是，使用该函数时，内核会在处理中断时自动调用 `InterruptMask()` 。因此，你应该在中断处理线程中调用 `InterruptUnmask()` 清除中断。
