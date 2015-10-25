title: 关于Looper的瞎扯蛋
date: 2015-8-26 06:42:48
tags:
---
#关于Looper的瞎扯蛋

知乎有这么一个问题

[Android中为什么主线程不会因为Looper.loop()里的死循环卡死？](http://www.zhihu.com/question/34652589)

---

这问题本身其实很简单,对Android有一定了解的都会学习Handler Looper MessageQueue相关的知识,面试的时候我也喜欢问别人这个问题.



        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            msg.target.dispatchMessage(msg);

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
        
        
要我简单点回答的话,我觉得答案就是 阻塞队列 `BlockingQueue`

这里比较有意思的这个问题里的有个些答案让我很意外


    著作权归作者所有。
    商业转载请联系作者获得授权，非商业转载请注明出处。
    作者：TracyB
    链接：http://www.zhihu.com/question/34652589/answer/59722754
    来源：知乎
    
    
    epoll+pipe，有消息就依次执行，没消息就block住，让出CPU，等有消息了，epoll会往pipe中写一个字符，把主线程从block状态唤起，主线程就继续依次执行消息，怎么会死循环呢…


这个逻辑我猜大概是这样

为什么不会死循环 --> 这里使用了阻塞队列 --> 阻塞队列底层调度如何实现 --> epoll

我猜这人大概是做Framework层 或 ROM开发的, 一般Android程序员恐怕不会从这个角度去想问题.

---
ps.顺便一提

Looper中通过 `ThreadLocal` 来为每个线程分配隔离的 Looper对象

    
    static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
    

关于ThreadLocal,以前都是当做Java特性的来理解这个东西的,无聊看了下源代码

实现机制其实非常简洁,我觉得我以前看过的书上 对 ThreadLocal的介绍&分析真是浪费时间, 核心代码也没几行, 看源码5分钟就能理解的东西,书上啰啰嗦嗦的讲上N页


    public T get() {
        // Optimized for the fast path.
        Thread currentThread = Thread.currentThread();
        Values values = values(currentThread);
        if (values != null) {
            Object[] table = values.table;
            int index = hash & values.mask;
            if (this.reference == table[index]) {
                return (T) table[index + 1];
            }
        } else {
            values = initializeValues(currentThread);
        }

        return (T) values.getAfterMiss(this);
    }

    public void set(T value) {
        Thread currentThread = Thread.currentThread();
        Values values = values(currentThread);
        if (values == null) {
            values = initializeValues(currentThread);
        }
        values.put(this, value);
    }
    
    Values values(Thread current) {
        return current.localValues;
    }


几个关键方法源代码一看,每个Thread会持有一个localValues对象,set的时候,对象被直接塞给了localValues
localValues是一个类似HashMap的数据结构,ThreadLocal对象自身做key


简单明了,看代码根本不会产生任何理解上的误差,以前像记概念一样看这个东西真是犯傻.


---

ps2.顺便我就想到了volatile

这个玩意也是一直是当做概念去记的. 比如JAVA核心技术上用了一大段晦涩的文字来描述这是个什么玩意,印象中还在其它书上看到更复杂的解释.

实际上 一旦更深入的划分 `内存` 这个概念 , 这个东西就很好理解了,而且不会再记错用错.

CPU寄存器中的cache是内存, JVM的 线程工作内存 也是内存.

