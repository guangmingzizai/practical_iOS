# AutoreleasePool底层实现

功能特性：

* 与线程绑定
* 可嵌套
* 单个添加对象
* 整体释放对象

## 数据结构

AutoreleasePool本身没有单独的内存结构，它是通过以AutoreleasePoolPage为结点的双向链表来实现的。

![AutoreleasePoolPage](http://blog.leichunfeng.com/images/AutoreleasePoolPage.png)

```c++
class AutoreleasePoolPage
{
    magic_t const magic; // 用于完整性校验
    id *next;
    pthread_t const thread;
    AutoreleasePoolPage * const parent;
    AutoreleasePoolPage *child;
    uint32_t const depth;
    uint32_t hiwat; // high water mark
}
```



## 参考资料

* [autorelease源码](https://opensource.apple.com/source/objc4/objc4-750/runtime/NSObject.mm.auto.html)
* [Autorelease Pool的实现原理](http://blog.leichunfeng.com/blog/2015/05/31/objective-c-autorelease-pool-implementation-principle/)
* [深入理解 Autorelease](https://juejin.im/post/5a66e28c6fb9a01cbf387da1#heading-31)

## 问题

### AutoreleasePool和RunLoop的关系？

本质上没有关系，只不过系统在主线程的RunLoop上添加了三个Observer，即将进入RunLoop时以最高优先级调用`objc_autoreleasePooLPush`，在休眠时调用`objc_autoreleasePop`和`objc_autoreleasePoolPush`，在退出RunLoop时以最低优先级调用`objc_autoreleasePoolPop`。

![RunLoop](https://blog.ibireme.com/wp-content/uploads/2015/05/RunLoop_1.png)

详见：https://blog.ibireme.com/2015/05/18/runloop/#autorelease

### AutoreleasePool和线程的关系？

Cocoa应用中的每个线程都维护了自己的autorelease pool栈，每一个AutoreleasePool只对应一个线程。

AutoreleasePoolPage通过TLS（Thread Local Storage，线程局部存储）与线程关联。

### 什么是AutoreleasePool栈？AutoreleasePool底层是如何工作的？

http://matteogobbi.github.io/blog/2014/09/28/autorelease-under-the-hood/

### 什么情况下需要手动创建AutoreleasePool？

* 如果你编写的程序不是基于Cocoa框架的，比如说命令行工具；
* 如果你编写的循环中创建了大量的临时对象；
* 如果你创建了一个辅助线程。(iOS7以后，AutoreleasePool会自动创建)

### 子线程默认不开启RunLoop，如果不手动创建AutoreleasePool，Autorelease对象会如何处理？

iOS7以后，当调用`autorelease`方法时，系统会自动创建AutoreleasePool。`AutoreleasePoolPage::push`和`AutoreleasePoolPage::autorelease`最终都会调用AutoreleasePoolPage的`autoreleaseFast`方法：

```c++
static inline id *autoreleaseFast(id obj)
{
    AutoreleasePoolPage *page = hotPage();
    if (page && !page->full()) {
        return page->add(obj);
    } else if (page) {
        return autoreleaseFullPage(obj, page);
    } else {
        return autoreleaseNoPage(obj);
    }
}
```

区别只在于`push`插入的是`POOL_SENTINEL`，而`autorelease`插入的是实际的对象地址。

### @autoreleasepool是如何实现的？

```c++
struct __AtAutoreleasePool {
    __AtAutoreleasePool() {atautoreleasepoolobj = objc_autoreleasePoolPush();}
    ~__AtAutoreleasePool() {objc_autoreleasePoolPop(atautoreleasepoolobj);}
    void * atautoreleasepoolobj;
};
{
	__AtAutoreleasePool __autoreleasepool;
}
```

