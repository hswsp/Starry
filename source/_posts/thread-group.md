---
title: Java ThreadGroup
description: thread-group
author: Starry
tags:
  - Multithreading
categories:
  - Java
  - Multithreading
date: 2022-11-06 00:53:44
index_img: https://th.bing.com/th/id/OIP.rgXrFs9LKjgO3su29sW8wQHaEo?pid=ImgDet&rs=1
banner_img: https://th.bing.com/th/id/R.433739b74906eee73d8ffbd0ae91ba4c?rik=hUlewY8BWwkbvw&riu=http%3a%2f%2fwww.gufengjia.com%2fd%2ffile%2fimages%2f201907010010%2f13-1P612143K0.jpg&ehk=ETEcc7MrzIM%2bRjhh5C%2fzVfYJIYNw5wXF7r6bWGCtigY%3d&risl=&pid=ImgRaw&r=0
---

# 线程组概念理解

在java的多线程处理中有线程组ThreadGroup的概念，**ThreadGroup是为了方便线程管理出现了，可以统一设定线程组的一些属性，比如setDaemon，设置未处理异常的处理方法，设置统一的安全策略等等**；也可以通过线程组方便的获得线程的一些信息。

每一个ThreadGroup都可以包含一组的子线程和一组子线程组，在一个**进程中线程组是以树形的方式存在**，通常情况下根线程组是system线程组。system线程组下是main线程组，默认情况下第一级应用自己的线程组是通过main线程组创建出来的。

我们可以通过下面代码片段看下一个简单的java application中线程组的情况：

```java
package cn.outofmemory.concurrent;
 
public class ThreadGroupDemo {
	public static void main(String[] args) {
		printGroupInfo(Thread.currentThread());
		
		Thread appThread = new Thread(new Runnable(){
			@Override
			public void run() {
				for (int i=0;i<5;i++) {
					System.out.println("do loop " + i);
				}
			}
		});
		appThread.setName("appThread");
		appThread.start();
		printGroupInfo(appThread);
	}
	
	static void printGroupInfo(Thread t) {
		ThreadGroup group = t.getThreadGroup();
		System.out.println("thread " + t.getName() + " group name is " 
				+ group.getName()+ " max priority is " + group.getMaxPriority()
				+ " thread count is " + group.activeCount());
		
		ThreadGroup parent=group;
		do {
			ThreadGroup current = parent;
			parent = parent.getParent();
			if (parent == null) {
				break;
			}
 
			System.out.println(current.getName() + "'s parent is " + parent.getName());
		} while (true);
		System.out.println("--------------------------");
	}
}
```

这段代码打印结果如下：

```bash
thread main group name is main max priority is 10 thread count is 1
main's parent is system
--------------------------
thread appThread group name is main max priority is 10 thread count is 2
main's parent is system
--------------------------
do loop 0
do loop 1
do loop 2
do loop 3
do loop 4
```

在创建线程时可以通过构造函数指定其所在的线程组，如下代码：

```java
ThreadGroup group=new ThreadGroup("worker");
Thread thread=new Thread(group,"the first thread of group");
```

通常情况下我们创建线程时可能不设置线程组，这时候**创建的线程会和创建该线程的线程在一个组里面**。

# 线程组实践

下面通过实例演示如何使用ThreadGroup，该实例中设定了线程组的Daemon属性，设置了线程组内线程的最大优先级，通过继承ThreadGroup的方式，重写了该线程组对于未处理异常的处理逻辑。

```java
package cn.outofmemory.concurrent;
 
import java.net.SocketException;
 
public class ThreadGroupDemo2 {
	public static void main(String[] args) {
		ThreadGroup spiderGroup = new SpiderThreadGroup("spiderGroup");
		//可以统一设定线程是否为守护线程
		spiderGroup.setDaemon(true);
		
		//可以设置线程组内的最大优先级
		spiderGroup.setMaxPriority(Thread.NORM_PRIORITY);
		
		//初始化线程
		Thread spiderThread = new Thread(spiderGroup, new Runnable() {
			@Override
			public void run() {
				throw new RuntimeException(new SocketException());
			}
			
		});
		
		//启动线程
		spiderThread.start();
	}
 
 
	/**
	 * 此类从ThreadGroup类继承重写了其uncaughtException方法，对于SocketException进行了特殊处理
	 * @author outofmemory.cn
	 *
	 */
	static class SpiderThreadGroup extends ThreadGroup {
		public SpiderThreadGroup(String name) {
			super(name);
		}
 
		public void uncaughtException(Thread t, Throwable e) {
			if (e.getCause() instanceof SocketException) {
				System.out.println("socket exception should be process");
			} else {
				super.uncaughtException(t, e);
			}
		}
	}
}
```

上面的实例代码会输出如下内容：

```bash
socket exception should be process
```

说明未处理异常已经被统一处理了。

下面我们通过另外一个示例演示如何通过线程组，方便的获得应用中一共有多少个活动线程，并打印这些活动线程的名字。

```java
package cn.outofmemory.concurrent;
 
public class ThreadDemo3 {
	public static void main(String[] args) {
		ThreadGroup g = Thread.currentThread().getThreadGroup();
		while (g != null) {
			ThreadGroup temp = g.getParent();
			if (temp == null) {
				break;
			}
			g = temp;
		}
		
		//现在g就是跟线程组
		System.out.println("active count is " + g.activeCount());
		
		Thread[] all = new Thread[g.activeCount()];
		g.enumerate(all);
		for (Thread t : all) {
			System.out.println(t.getName());
		}
	}
}
```

按理说我只有一个main方法，没有任何自定义线程，那么会输出几个线程呢？

下面是输出结果：

```bash
active count is 5
Reference Handler
Finalizer
Signal Dispatcher
Attach Listener
main
```

一共有5个线程，这5个线程除了main是我们自己代码run所在的线程，其他都是虚拟机启动的线程。

# 线程组和线程池

线程组和线程池是两个不同的概念，他们的作用完全不同，**线程组是为了方便线程的管理，线程池是为了管理线程的生命周期，复用线程，减少创建销毁线程的开销。**

 在构建线程池时，ThreadPoolExcutor的构造方法中，生成线程工厂的参数ThreadFactory接口的实现类**DefaultThreadFactory中有个属性，ThreadGroup**

```java
 public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

```java
public class DefaultThreadFactory implements ThreadFactory {
 
    private static final AtomicInteger poolId = new AtomicInteger();
 
    private final AtomicInteger nextId = new AtomicInteger();
    private final String prefix;
    private final boolean daemon;
    private final int priority;
    protected final ThreadGroup threadGroup;
    ......
}
```
