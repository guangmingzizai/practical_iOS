# DispatchQueue QoS/Priority探究

本文只探讨一个问题：QoS/Priority到底代表什么意思？执行机会更多？还是低优先级线程必须等待所有Running/Ready状态的高优先级线程全部执行完毕？

我们知道，在iOS的各种锁机制中，自旋锁的性能是最高的，但由于潜在的优先级反转问题，已经被弃用了，详情请见：[不再安全的 OSSpinLock](https://blog.ibireme.com/2016/01/16/spinlock_is_unsafe_in_ios/)。此篇不讨论锁的问题，而是讨论一下指出自旋锁问题的[swift-dev邮件列表](https://lists.swift.org/pipermail/swift-dev/Week-of-Mon-20151214/000372.html)的内容，个人认为邮件中的说法并不妥当。

邮件原文：

```
The iOS scheduler maintains several different priority levels / QoS classes: background, utility, default, user-initiated, user-interactive. If any thread in a higher class is runnable then it will always run before every thread in lower classes. A thread's priority will never decay down into a lower class. (I am told that I/O throttling has similar effects, but I don't know the details there.)

This breaks naïve spinlocks like OSSpinLock. If a lower priority thread acquires the lock and is scheduled out, and then enough high-priority threads spin on the lock, then the low-priority thread will be starved and will never run again.

This is not a theoretical problem. libobjc saw dozens of livelocks against its internal spinlocks until we stopped using OSSpinLock.
```

本人理解（欢迎拍砖）：只要存在Running/Ready状态的高优先级线程，那么低优先级线程将永远不会被执行。

看了几篇讨论自旋锁问题的文章，意思也都是说，自旋锁忙等的机制导致高优先级线程霸占时间片，导致低优先级线程任务无法完成，从而无法释放锁。

之前并没有深入思考这个问题，今天在研究一个问题时，再次看到了这封邮件，但对文中的说法产生了怀疑，感觉这并不符合正常的系统设计思路，不太相信苹果会设计出如此脑残的机制。而且根据QoS的定义，丝毫看不出高优先级会阻塞低优先级，应该只是执行机会更多而已。

于是做了以下实验：

```swift
class HighBlockLow {
    
    static func doAction() {
        startLow()
        startHigh()
    }
    
    private static func startHigh() {
        DispatchQueue.global(qos: DispatchQoS.QoSClass.userInitiated).async {
            doWork(priority: "userInitiated", sleep: false)
        }
    }
    
    private static func startLow() {
        DispatchQueue.global(qos: DispatchQoS.QoSClass.utility).async {
            doWork(priority: "utility", sleep: false)
        }
    }
    
    private static func doWork(priority: String, sleep: Bool) {
        print("\(priority) started")
        var count: UInt32 = 0
        while true {
            count += 1
            count %= UInt32.max
            if sleep || count % 10000000 == 0 {
                print("\(priority): \(count)")
            }
            if sleep {
                Thread.sleep(forTimeInterval: 0.1)
            }
        }
    }
    
}
```

实验结果证明，两个死循环的线程，高优先级线程并不会导致低优先级线程无法执行的情况，只是高优先级线程获得的执行机会更多而已，但是差别并没有那么恐怖，以上代码的差别在2倍以内，其它实验也在5倍以内，甚至出现过低优先线程快于高优先级线程的情况。

**哪位朋友能指出如何能让低优先的线程无法执行？**

