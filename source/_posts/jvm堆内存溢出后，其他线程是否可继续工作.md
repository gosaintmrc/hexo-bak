---
title: jvm堆内存溢出后，其他线程是否可继续工作
date: 2019-01-31 15:32:00
categories: 
- Java基础
tags:  jvm
---

最近网上出现一个美团面试题：“一个线程OOM后，其他线程还能运行吗？”。我看网上出现了很多不靠谱的答案。这道题其实很有难度，涉及的知识点有jvm内存分配、作用域、gc等，不是简单的是与否的问题。

由于题目中给出的OOM，java中OOM又分很多类型；比如：堆溢出（“java.lang.OutOfMemoryError: Java heap space”）、永久带溢出（“java.lang.OutOfMemoryError:Permgen space”）、不能创建线程（“java.lang.OutOfMemoryError:Unable to create new native thread”）等很多种情况。

本文主要是分析堆溢出对应用带来的影响。

先说一下答案，答案是还能运行。

代码如下：

```java
public class JvmThread {
 
 
    public static void main(String[] args) {
        new Thread(() -> {
            List<byte[]> list = new ArrayList<byte[]>();
            while (true) {
                System.out.println(new Date().toString() + Thread.currentThread() + "==");
                byte[] b = new byte[1024 * 1024 * 1];
                list.add(b);
                try {
                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
 
        // 线程二
        new Thread(() -> {
            while (true) {
                System.out.println(new Date().toString() + Thread.currentThread() + "==");
                try {
                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}

```

结果展示：

```java
Wed Nov 07 14:42:18 CST 2018Thread[Thread-1,5,main]==
Wed Nov 07 14:42:18 CST 2018Thread[Thread-0,5,main]==
Wed Nov 07 14:42:19 CST 2018Thread[Thread-1,5,main]==
Wed Nov 07 14:42:19 CST 2018Thread[Thread-0,5,main]==
Exception in thread "Thread-0" java.lang.OutOfMemoryError: Java heap space
	at com.gosaint.util.JvmThread.lambda$main$0(JvmThread.java:21)
	at com.gosaint.util.JvmThread$$Lambda$1/521645586.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)
Wed Nov 07 14:42:20 CST 2018Thread[Thread-1,5,main]==
Wed Nov 07 14:42:21 CST 2018Thread[Thread-1,5,main]==
Wed Nov 07 14:42:22 CST 2018Thread[Thread-1,5,main]==

```

JVM启动参数设置：

![](jvm堆内存溢出后，其他线程是否可继续工作\1.png)

![2](jvm堆内存溢出后，其他线程是否可继续工作\2.png)

上图是JVM堆空间的变化。我们仔细观察一下在14:42:05~14:42:25之间曲线变化，你会发现使用堆的数量，突然间急剧下滑！这代表这一点，当一个线程抛出OOM异常后，它所占据的内存资源会全部被释放掉，从而不会影响其他线程的运行！

讲到这里大家应该懂了，此题的答案为一个线程溢出后，进程里的其他线程还能照常运行。注意了，这个例子我只演示了堆溢出的情况。如果是栈溢出，结论也是一样的，大家可自行通过代码测试。

总结：其实发生OOM的线程一般情况下会死亡，也就是会被终结掉，该线程持有的对象占用的heap都会被gc了，释放内存。因为发生OOM之前要进行gc，就算其他线程能够正常工作，也会因为频繁gc产生较大的影响。

参考：https://mp.weixin.qq.com/s/TXu6LOAN2i9oAOQaLf7saQ
--------------------- 