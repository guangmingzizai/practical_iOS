# iOS知识点

## 杂记

* 堆地址小于栈地址，iOS中一个进程的栈区内存为1M，Mac为8M。
* `nm`命令可以列表文件中的符号信息，输入为目标文件、静态库、动态库等。
* `load`方法是根据函数地址直接调用，而`initialize`方法是通过`objc_msgSend`调用的。
* 根类：NSObject、NSProxy。

### TableViewCell高度自适应

设置`tableView.rowHeight = UITableViewAutomaticDimension`或在 `- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath` 方法中返回 `UITableViewAutomaticDimension`，同时设置`estimatedRowHeight`，那么tableView就会自动计算cell的高度(section header or footer 同理)。

如果cell内部使用了AutoLayout（contentView上面添加了约束），高度通过AutoLayout计算。否则，调用cell的 `- (CGSize)sizeThatFits:(CGSize)size`方法。

### 参考资源

* [什么是代码区、常量区、静态区（全局区）、堆区、栈区](https://blog.csdn.net/u014470361/article/details/79297601)
* [内存对齐](https://www.cnblogs.com/jijiji/p/4854581.html)
* [Tagged Pointer](http://www.cocoachina.com/ios/20150918/13449.html)
* [OC对象的内存结构](https://www.cnblogs.com/chrisbin/articles/6486906.html)
* [每个程序员都应该了解的 CPU 高速缓存](https://www.oschina.net/translate/what-every-programmer-should-know-about-cpu-cache-part2?print)
* [UIViewController的生命周期](https://juejin.im/post/5a706cf05188257323357286)

## 调试
### p/po/expression
`expression`命令执行一个表达式，并将表达式返回的结果输出，是LLDB调试命令中最重要的命令，也是常用的`p`和`po`命令的鼻祖。
`expression`命令有两个功能：执行表达式、输出表达式的值。如：
```shell
(lldb) expression -- self.view.backgroundColor = [UIColor yellowColor] // 改变颜色

(lldb) expression -- (void)[CATransaction flush] // 刷新屏幕

(lldb) expression -- self.view
```
由`expression`命令衍生出了几个常用的命令：`p`、`print`、`e`和`call`，这几个命令其实就是`expression --`的别名。

#### po
OC里所有的对象都是用指针表示的，打印出来的是对象的指针，而不是对象本身。可以用`-o`选项来打印对象本身。
`po`其实就是`expression -o --`的别名。

## Blocks

### 数据结构

![block_data_structure](http://upload-images.jianshu.io/upload_images/1363078-9e217f323d2c28a7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

结构体定义：

```c
struct Block_descriptor {
   unsigned long int reserved;
   unsigned long int size;
   void (*copy)(void *dst, void *src); 
   void (*dispose)(void *);
};
struct Block_layout { 
    void *isa; 
    int flags; 
    int reserved;
    void (*invoke)(void *, ...);
    struct Block_descriptor *descriptor; 
   /* Imported variables. */
};
```

block实际上由6部分组成：

1. isa指针，所有对象都有该指针。
2. flags，用于按位表示一些block附加信息。
3. reserved, 保留变量。
4. invoke, 函数指针，指向具体的block实现的函数调用地址。
5. descriptor, 表示该block的附加描述信息，主要是size的大小，以及copy和dispose函数指针。
6. variables，capture过来的变量，block能够访问它外部的局部变量，就是因为将这些变量复制到了结构体中。

### ARC，MRC 对Block类型的影响

 三种类型：`__NSMallocBlock__`、`__NSStackBlock__`和`__NSGlobalBlock__`。

**在Block中，如果只使用全局或静态变量或者不使用外部变量，那么Block块的代码会存储在全局区。**
在ARC中

- 如果使用外部变量，Block块的代码会存储在堆区。

在MRC中

- 如果使用外部变量，Block块的代码会存储在栈区。

**Block默认情况下不能修改外部变量，只能读取外部变量。**
在ARC中

- 外部变量在堆中，这个变量在Block块内与在Block块外地址相同。
- 外部变量在栈中，这个变量会被copy到为Block代码块所分配的堆中。

在MRC中

- 外部变量在堆中，这个变量在Block块内与在Block块外地址相同。
- 外部变量在栈中，这个变量会被copy到为Block代码块所分配的**栈中**。

**如果需要修改外部变量，需要在外部变量前声明__block。**
在ARC中

- 外部变量存在堆中，这个变量在Block块内与Block块外地址相同。
- 外部变量存在栈中，这个变量会被转移到堆中，不是复制。

在MRC中

- 外部变量存在堆中，这个变量在Block块内与Block块外地址相同。
- 外部变量存在栈中，这个变量在Block块内与Block块外地址相同。

### __block的原理

将栈上用__block修饰的自动变量**封装成一个结构体**，让其在堆上创建，以方便从栈上或堆上访问和修改同一份数据。

对于block外的变量引用，block默认是将其复制到其数据结构中来实现访问的，如下图所示:

![block_outter_varible](https://blog-image-1252287090.cos.ap-beijing.myqcloud.com/block-capture-1.jpg?q-sign-algorithm=sha1&q-ak=AKIDDitEeoFtMCCuwTk7Y264l878rTygcMEM&q-sign-time=1550374387;1550376187&q-key-time=1550374387;1550376187&q-header-list=&q-url-param-list=&q-signature=7cf09e9bbecb2dde44867855b903be96a3f8a97f&x-cos-security-token=8dc22d225c07eeefbb4bea1bf0d7c6cdc69a31a110001)



对于用__block修饰的外部变量引用，block是复制其引用地址来实现访问的，如下图所示：

![block_outter_variable__block](https://blog.devtang.com/images/block-capture-2.jpg)

对于如下代码:

```c
__block int a = 100;
void (^block)(void) = ^{
	a = 200;
	printf("%d\n", a);
};
```

编辑之后会生成类似如下代码：

```c
struct __Block_byref_a_0 {
  void *__isa;
__Block_byref_a_0 *__forwarding;
 int __flags;
 int __size;
 int a;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __Block_byref_a_0 *a; // by ref
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_a_0 *_a, int flags=0) : a(_a->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  __Block_byref_a_0 *a = __cself->a; // bound by ref

        (a->__forwarding->a) = 200;
        printf("%d\n", (a->__forwarding->a));
    }
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->a, (void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);}

static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);}

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
  void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};
int main(int argc, char const *argv[])
{
    __attribute__((__blocks__(byref))) __Block_byref_a_0 a = {(void*)0,(__Block_byref_a_0 *)&a, 0, sizeof(__Block_byref_a_0), 100};
    void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_a_0 *)&a, 570425344));
    ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);
    return 0;
}
```

### __weak引用

```objective-c
- (void)weakBlockTest {
    __weak void (^test)(void) = ^{
    	NSLog(@"Hello World!");
	};
	test();
}
```

以上代码的运行结果是什么？**正常输出**。

分析：以非\__strong引用创建block时，block在栈上，即__weak并不会影响block的生命周期。

### 自动从栈复制到堆的情况

* 调用block的copy方法
* 将block作为函数返回值返回
* 将block赋值给__strong修饰的变量
* 向Cocoa框架中含有usingBlock的方法或者GCD的API传递block参数时

*在函数内部创建的block，如果没有捕获自动变量，那么block存储在全局数据区，而不是栈上。*

### 参考资源

* https://blog.devtang.com/2013/07/28/a-look-inside-blocks/
* [A look inside blocks: Episode 1](http://www.galloway.me.uk/2012/10/a-look-inside-blocks-episode-1/)
* [A look inside blocks: Episode 2](http://www.galloway.me.uk/2012/10/a-look-inside-blocks-episode-2/)
* [A look inside blocks: Episode 3](http://www.galloway.me.uk/2013/05/a-look-inside-blocks-episode-3-block-copy/)
* [Objective-C编译成C++代码报错](https://www.jianshu.com/p/43a09727eb2c)
* [探寻block的本质](https://juejin.im/post/5b0181e15188254270643e88)
* [Block用法和实现原理](https://www.jianshu.com/p/d28a5633b963)



## Autorelease

Autorelease pool implementation:

> A thread's autorelease pool is a stack of pointers. 
> Each pointer is either an object to release, or POOL_SENTINEL which is an autorelease pool boundary.
> A pool token is a pointer to the POOL_SENTINEL for that pool. When the pool is popped, every object hotter than the sentinel is released.
> The stack is divided into a doubly-linked list of pages. Pages are added and deleted as necessary. 
> Thread-local storage points to the hot page, where newly autoreleased objects are stored.

```c++
class AutoreleasePoolPage {
    magic_t const magic;
    id *next;
    pthread_t const thread;
    AutoreleasePoolPage * const parent;
    AutoreleasePoolPage *child;
    uint32_t const depth;
    uint32_t hiwat;
};
```

参考：https://juejin.im/post/5a66e28c6fb9a01cbf387da1



## KVO

**iOS用什么方式实现对一个对象的KVO？**

当一个对象使用了KVO监听，iOS系统会修改这个对象的isa指针，改为指向一个全新的通过Runtime动态创建的子类，子类拥有自己的set方法实现，set方法实现内部会顺序调用**willChangeValueForKey方法、原来的setter方法实现、didChangeValueForKey方法，而didChangeValueForKey方法内部又会调用监听器的observeValueForKeyPath:ofObject:change:context:监听方法。**

**如何手动触发KVO？**

被监听的属性的值被修改时，就会自动触发KVO。如果想要手动触发KVO，则需要我们自己调用**willChangeValueForKey和didChangeValueForKey**方法即可在不改变属性值的情况下手动触发KVO，并且这两个方法缺一不可。

## KVC

KVC是NSObject类的扩展（OC有显式的`NSKeyValueCoding`类别），因此纯Swift类和结构体是不支持KVC的。

### 设置过程

当调用`setValue：属性值 forKey：@”name“`的代码时，底层的执行机制如下：

- 程序优先调用`set<Key>:属性值`方法，代码通过`setter`方法完成设置。**注意，这里的<key>是指成员变量名，首字母大小写要符合KVC的命名规则，下同**
- 如果没有找到`setName：`方法，KVC机制会检查`+ (BOOL)accessInstanceVariablesDirectly`方法有没有返回YES，默认该方法会返回YES，如果你重写了该方法让其返回NO的话，那么在这一步KVC会执行`setValue：forUndefinedKey：`方法，不过一般开发者不会这么做。所以KVC机制会搜索该类里面有没有名为`_<key>`的成员变量，无论该变量是在类接口处定义，还是在类实现处定义，也无论用了什么样的访问修饰符，只要存在以`_<key>`命名的变量，KVC都可以对该成员变量赋值。
- 如果该类即没有`set<key>：`方法，也没有`_<key>`成员变量，KVC机制会搜索`_is<Key>`的成员变量。
- 和上面一样，如果该类即没有`set<Key>：`方法，也没有`_<key>`和`_is<Key>`成员变量，KVC机制再会继续搜索`<key>`和`is<Key>`的成员变量。再给它们赋值。
- 如果上面列出的方法或者成员变量都不存在，系统将会执行该对象的`setValue：forUndefinedKey：`方法，默认是抛出异常。

如果开发者想让这个类禁用KVC，那么重写`+ (BOOL)accessInstanceVariablesDirectly`方法让其返回NO即可，这样的话如果KVC没有找到`set<Key>:`属性名时，会直接用`setValue：forUndefinedKey：`方法。

### 参考

[详解KVC](https://juejin.im/entry/5a7c4950f265da4e7071b5c9)

[KVC-KVO](https://github.com/leejayID/KVC-KVO)

## Category

定义：

```c
struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods; // 对象方法
    struct method_list_t *classMethods; // 类方法
    struct protocol_list_t *protocols; // 协议
    struct property_list_t *instanceProperties; // 属性
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};
```

问： Category的实现原理，以及Category为什么只能加方法不能加属性?

答：分类的实现原理是将category中的方法，属性，协议数据放在category_t结构体中，然后将结构体内的方法列表拷贝到类对象的方法列表中。 Category可以添加属性，但是并不会自动生成成员变量及set/get方法。因为category_t结构体中并不存在成员变量。通过之前对对象的分析我们知道成员变量是存放在实例对象中的，并且编译的那一刻就已经决定好了。而分类是在运行时才去加载的。那么我们就无法再程序运行时将分类的成员变量中添加到实例对象的结构体中。因此分类中不可以添加成员变量。

问：Category中有load方法吗？load方法是什么时候调用的？load 方法能继承吗？

答：Category中有load方法，load方法在程序启动装载类信息的时候就会调用。load方法可以继承。调用子类的load方法之前，会先调用父类的load方法

问：load、initialize的区别，以及它们在category重写的时候的调用的次序。

答：区别在于调用方式和调用时刻 调用方式：load是根据函数地址直接调用，initialize是通过objc_msgSend调用 调用时刻：load是runtime加载类、分类的时候调用（只会调用1次），initialize是类第一次接收到消息的时候调用，每一个类只会initialize一次（父类的initialize方法可能会被调用多次）

调用顺序：先调用类的load方法，先编译那个类，就先调用load。在调用load之前会先调用父类的load方法。分类中load方法不会覆盖本类的load方法，先编译的分类优先调用load方法。initialize先初始化父类，之后再初始化子类。如果子类没有实现+initialize，会调用父类的+initialize（所以父类的+initialize可能会被调用多次），如果分类实现了+initialize，就覆盖类本身的+initialize调用。

## Runtime

OC将尽可能多的决策从编译、链接推迟到了运行时，只要有可能，OC就会动态处理。这就意味着，处理编译器之外，还需要一个运行时系统来执行编译之后的代码。

### 类继承体系

![class_hierarchy](https://upload-images.jianshu.io/upload_images/449095-3e972ec16703c54d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/598/format/jpeg)



### 消息转发流程

![message_forward](https://upload-images.jianshu.io/upload_images/1395150-aa440fea34187cd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/588)

### @dynamic

```objective-c
@dynamic propertyName;
```

which tells the compiler that the methods associated with the property will be provided dynamically.

### 知识点

* `objc_class`会缓存最近访问方法，以提高后续调用的效率。
* meta-class是一个类对象的类。所有meta-class的isa指向基类的meta-class，基类的meta-class的isa指针指向自己。
* 类本身也是一个对象

### 参考

* http://www.codeceo.com/article/objective-c-runtime-class.html
* Objective-C Runtime Programming Guide
* [iOS runtime探究](https://www.jianshu.com/p/eac6ed137e06)

## RunLoop

RunLoop是线程的核心基础设施之一，一个RunLoop就是一个事件处理循环，可以用来计划任务和协调输入事件的接受。RunLoop的目的是当有任务时保持线程活跃，当没有工作时使线程休眠。

RunLoop的管理不是完全自动的，你需要在自己的线程代码中适时地启动run loop并响应输入事件。

### 参考

* [Mach原语：一切以消息为媒介](https://www.jianshu.com/p/284b1777586c)
* [深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)
* [Threading Programming Guide - Run Loops](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)

## 并行编程

### 线程同步

#### 锁的实现方式

* **阻塞**：如果锁对象被其他线程所持有，那么请求访问的线程就会被加入到等待队列中，因而被阻塞。这就意味着被阻塞的线程放弃了时间片，调度器会将CPU让给下一个执行的的线程。**当锁可用的时候**，调度器会得到通知，然后根据情况将线程从等待队列取出来，并重新调度。

* **忙等**：线程不放弃CPU时间片，而是继续重复的尝试访问所对，直到锁可用。

* **阻塞与忙等的对比**：当锁只持续短短几个周期的时候，阻塞会带来性能问题，因为至少会消耗两次上下文切换的时间。因此如果锁的持续时间短，应该采用忙等形式的锁对象，反之就采用阻塞形式的锁对象。

#### 互斥体(lck_mtx_t)（阻塞）

- 互斥体其实就是一个内核中的一个不同的变量，通常是机器字节大小的整数，但是必须要求硬件能对这些变量进行原子操作。
- 原子的意思就是：对互斥体的操作不能打断，即使是硬件中断也不能打断。

#### 信号量(semaphore_t)（阻塞）

信号量在初始化的时候可以设置一个大于零的初始值。信号量包含两个操作：一个是+1操作，一个是-1操作，当值大于0表示锁可用，当值小于等于0的时候表示锁不可用。互斥体可以看做是初始值为1的信号量。

#### 自旋锁(hw_lock_t)（忙等）

一种采用忙等形式的锁。

#### 读写锁(hw_lock_t)（阻塞）

当多个线程对资源只做只读的操作，这种情况下这些线程并不会相互影响。为了提高效率，读写锁就应运而生了。读写锁能够区分是读访问还是写访问，对个读者可以同时持有锁，但一次只能一个写者持有锁。

问：`OSSpinLock`有什么问题？

存在优先级翻转的风险。iOS调度器维护了几个优先级/QoS类别：background、utility、default、user-initiated和user-interactive。如果有高优先级的线程可以运行，那么高优先级的线程将始终先于低优先级线程，且线程的优先级绝对不会降低。这样就破坏了spinLock的运行机制，spinLock采取简单的循环等待，本来是简单场景效率最高的同步方式，但是在这种情况下，如果一个低优先级的线程持有锁，当一个高优先级的线程在锁上自旋时，低优先级的线程将永远不会被执行，以致陷入死锁状态。

> 根据实验，高优先级线程并不会导致低优先级线程无法执行，只是高优先级线程被执行的机会更高而已。

[serious scheduling problems that render this a bad idea on macOS and totally unusable on iOS](https://lists.swift.org/pipermail/swift-dev/Week-of-Mon-20151214/000372.html)

### Grand Center Dispatch

GCD基于线程池模式，相对于直接创建线程这样的高耗方式，可以获得更高的性能和更低的执行延迟。

Task -> Queue -> Thread.

hyperthreading: 把一个物理CPU虚拟为两个逻辑CPU，允许单个处理器在同一时刻并行地抓取和执行两个独立的代码流。

### 互斥访问

需要注意一个性能问题：闭包捕获过程中可能的动态堆分配，可能导致性能下降10倍。

Swift目前在语言层面没有提供线程和并发相关的支持，因此需要使用`Dispatch`之类的异步库。

[“Concurrency” proposal in the Swift repository](https://github.com/apple/swift/blob/c760f6dfbf0179e9aff90f7bf7375f3af5331318/docs/proposals/Concurrency.rst).

### 互斥方法比较

`DispatchQueue.sync`是最慢的方式之一，由于不可避免的闭包捕获问题，比其它方式慢一个数量级。

除非Swift可以优化non-escaping闭包到栈上，否则唯一避免堆分配的方式是确保它们是内联的-很不幸，这在跨越模块边界的情况下是不可能的，这使得`DispatchQueue.sync`不必要地缓慢。

`objc_sync_enter/objc_sync_exit`，比libdispatch快一点儿（2-3倍），但依旧不够理想，而且依赖OC运行时（被限定在苹果的系统上）。

`OSSpinLock`是最快的，比`dispatch_sync`快20倍，由于存在优先级反转问题，几乎不可用。

`os_unfair_lock`只比`OSSpinLock`慢30%。但是，它不是先进先出的(FIFO)，互斥锁被给定了一个任意的等待时间（因此，"unfair"）。这意味着它不应该成为通用互斥锁的第一选择。

`pthread_mutex_lock/pthread_mutex_unlock`是唯一性能过关、可移植的选项。

`dispatch_semaphore_t`呢？速度比`pthread_mutex`还略快，但是并不适合做通用的互斥锁。

信号量容易引发一些错误和问题，最严重的就是[priority inversion](https://en.wikipedia.org/wiki/Priority_inversion)，和`OSSpinLock`的问题一样，但是更复杂，而且容易引起死锁。

也就是说，信号量是线程之间实现完成通知的好方式，但是由于设计上的复杂性和风险，应该被限定为发送完成通知信号，而不是互斥访问。

### 参考资源

[Multithreading](https://github.com/alspirichev/Multithreading)

[谈 iOS 的锁](https://juejin.im/post/5a8fdb1c5188257a856f55a8)

[Ultimate Grand Central Dispatch tutorial in Swift](https://theswiftdev.com/2018/07/10/ultimate-grand-central-dispatch-tutorial-in-swift/)

## 事件处理

### 参考

* [Using Responders and the Responder Chain to Handle Events](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events)

## DNS

DNS采用分级机制。

以`www.baidu.com`为例，完整的域名为`www.baidu.com.`，解析过程为`.` -> `com` -> `baidu` -> `www`。

### 跟踪

可以使用`dig`命令跟踪DNS的过程，`dig`是Domain Information Groper的简称，可以执行查询域名相关的任务。

选项：

+[no]question,+[no]answer,+[no]authority,+[no]stat,+short,+noall

### 参考资源

- [从dig命令理解DNS](https://blog.csdn.net/a583929112/article/details/66499771)

## IPV6兼容

客户端在IPV6环境下能正常运行，不要求服务器直接支持IPV6。

由于绝大多数服务器依然只支持IPV4，因此运营商在部署IPV6网络时会提供一个转化层，以实现IPV6-IPV4的访问。运作在IPV6环境下的客户端访问服务器时（需要把IPV4的地址合成为IPV6的地址），DNS64的服务器会首先尝试请求服务器的IPV6地址，如果服务器支持IPV6，会直接返回服务器的IPV6地址；否则，DNS64服务器会请求服务器的IPV4地址，然后将结果合成为IPV6地址之后返回给客户端。

![IPV6-IPV4](https://developer.apple.com/library/archive/documentation/NetworkingInternetWeb/Conceptual/NetworkingOverview/art/NAT64-DNS64-ResolutionOfIPv4_2x.png)

### 参考资源

- Supporting IPv6 DNS64/NAT64 Networks](https://developer.apple.com/library/archive/documentation/NetworkingInternetWeb/Conceptual/NetworkingOverview/UnderstandingandPreparingfortheIPv6Transition/UnderstandingandPreparingfortheIPv6Transition.html#//apple_ref/doc/uid/TP40010220-CH213-SW25)

## 精确计算

就像十进制数无法精确表示1/3一样，有些数字无法用二进制数精确表示，如0.1、0.2、0.3等。在一般的应用中，细微的误差是可以接受的，可以使用float、double进行运算。但在金融类应用中，往往需要精确计算。流行的编程语言都提供了类似$*Decimal*$的类专门来处理这种情景。

### 参考资源

* [Arithmetic on Strings in Objective C: Don’t use Floats — Use NSDecimalNumber!](https://medium.com/@brianclouser/arithmetic-on-strings-in-objective-c-don-t-use-floats-use-nsdecimalnumber-5a14939164e9)
* [THE FLOATING-POINT GUIDE](https://floating-point-gui.de/)
* [bc](x-man-page://bc)

## Transform

- [Transformation matrix](https://en.wikipedia.org/wiki/Transformation_matrix#Perspective_projection)
- [3D projection](https://en.wikipedia.org/wiki/3D_projection#Perspective_projection)

## Workspace/Project/Target/Scheme



## bounds/frame/transform

* https://ninjapro.wordpress.com/2016/08/23/understanding-uiviews-transform-property-part-1/
* https://ninjapro.wordpress.com/2016/08/27/understanding-uiviews-transform-property-final-part/
* https://en.wikipedia.org/wiki/Affine_transformation

## 图片处理

* [谈谈 iOS 中图片的解压缩](http://blog.leichunfeng.com/blog/2017/02/20/talking-about-the-decompression-of-the-image-in-ios/)
* [Advanced Imaging on iOS](https://www.slideshare.net/rsebbe/2014-cocoaheads-advimaging)

## Core Animation

- [CALayer Tutorial for iOS: Getting Started](https://www.raywenderlich.com/402-calayer-tutorial-for-ios-getting-started)

## Timer

Timers can be a surprisingly tricky tool to use correctly.

Deferred invocations and single fire timers are simple enough to get working but they vary between an unmaintainable anti-pattern that should never be used and a construct highly prone to subtle ordering problems between control and handler contexts.

Timers have a nasty tendency to *look* like they’re working but then break when barely related (or even *unrelated*) code changes slightly. 

Hoping that independent code will complete within a specific time period is the worst kind of [coupling](https://en.wikipedia.org/wiki/Coupling_(computer_programming)) (and is almost always ingoring a notification that could trigger it properly).

1. a timer should always be closely tied to an associated temporary resource
2. changes to either the timer or its associated temporary resource must resolve synchronously with the other (even when they don’t always *occur* synchronously)

Most problems around timers involve failure to meet one of these expectations.

Deferred invocations are sometimes useful for quickly probing and tesing scenarios during debug investigations but they are simply **too prone to problems to be safely used in a deployed program**.

I consider `after` to be unusable in deployed code due to its potential for causing maintenance problems; you can make it work but the result is highly fragile. Small changes to code *outside* the immediate scope of the timer can break its behavior. Worse: when it breaks, it might continue to *look* like it works and might pass your automated testing unless you hit the exact timing pattern required to cause problems.

A rescheduled timer is one where we needed to extend the deadline for the timer. An example is an idle timer (e.g. a sleep timer or a timeout timer). For an idle timer, each new activity should reset the timer to its full duration.

**Don’t use deferred invocations outside of debug investigations.**

[Design patterns for safe timer usage](https://www.cocoawithlove.com/blog/2016/07/30/timer-problems.html)

## Swift编译缓慢问题

Swift的类型检查非常耗费时间，下面的代码在Swift3.1上可能需要花费20秒，几乎所有的时间都用在了类型检查上：

```swift
let g: Double = -(1 + 2) + -(3 + 4) + 5
```

A quirk in the Swift type checker: the type checker will choose to resolve overloaded operators to non-generic overloads whenever possible.

In general, Swift’s complexity problem won’t be an issue unless you’re using two or more of the following features in a single expression:

- overloaded functions (including operators)
- literals
- closures without explicit types
- expressions where Swift’s default “every integer literal is an `Int` and every float literal is a `Double`” choice is wrong

Unlike other languages, `Double(x)` is not equivalent to `x as Double` in Swift. The constructor works more like another function and since it has multiple overloads on its parameter, it actually introduces another overloaded function into the search space (albeit at a different location in the expression). 

Ultimately, the `as` operator is the only way to cast without inserting further complexity. Fortunately, `as` binds tighter than most binary operators so it can be used without parentheses in most cases.

```swift
let x: Double = (1 + 2).negated() + (3 + 4).negated() + 5.negated()
```

Methods normally have fewer overloads than the common arithmetic operators and type inference with the `.` operator is often narrower than via free functions.

## Protocol Oriented Programming

Swift is protocol and value oriented.

[Protocols](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html) are used to define a “blueprint of methods, properties, and other requirements that suit a particular task or piece of functionality.”

Why the push towards protocols? When large class hierarchies are built, a lot of properties and functionality can get inherited. Developers tend to add (and keep adding) general features to the top of the hierarchy — mainly to high-level ancestor classes. Mid-level and leaf-level classes tend to be more specific and *not* be functionality receptacles. Ancestor classes tend to be buckets for new functionality; oftentimes they become “polluted” or “bloated” with too many, extra, extraneous, and/or unrelated features. Mid-level and leaf-level classes end up inheriting a lot more features than they need.

These concerns about OOP aren’t written in stone. A good developer can avoid many of the OOP pitfalls I just enumerated. It takes time, practice and experience. For example, developers can overcome the functional bloat problem by adding instances of other classes as members to the classes they’re building rather than inheriting from those other classes (i.e., composition over inheritance).

Modelling abstractions using classes relies on inheritance. Subclasses will have the ability of their super classes. A subclass can override that ability, add specific one. OOP works perfect till you need another ability from another class. There is no multiple inheritance in Swift. But protocols serve as blueprints rather than parents. A protocol models abstraction by describing what the implementation types shall implement. This is the key difference between OOP and Protocol OP. And two other pros over OOP are here:

- Types can conform more than one protocol. This brings perfect flexibility.
- Protocols also can be extended and adopted by classes, structs and enums.

> Many object-oriented programming languages are plagued with limitations surrounding the resolution of ambiguous extension definitions. Swift handles this quite elegantly through protocol extensions by allowing the programmer to take control where the compiler falls short.

https://www.appcoda.com/protocol-oriented-programming/

https://www.appcoda.com/pop-vs-oop/

## 内存

内存结构：

![memory_struture](http://upload-images.jianshu.io/upload_images/4886200-482eda33a50282b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

BSS是英文block started by symbol的简称，通常是指用来存放程序中未初始化的全局变量的一块内存区域，在程序载入时由内核清0。

分页机制允许进程的物理内存空间可以是非连续的，方便系统更灵活地管理内存。就像商场的储物柜一样，当我们要存放东西时，商场不是把所有顾客的东西都堆在一起，而且给顾客分配一个或多个小格子，小格子（页）是最小的分配单位，这样方便商场管理。跟内存的问题一样，页内部可能会有空闲。

[探寻iOS内存分配](http://www.cocoachina.com/ios/20170908/20511.html)

## 函数式编程

[Functor、Applicative 和 Monad](http://blog.leichunfeng.com/blog/2015/11/08/functor-applicative-and-monad/)



## 编程风格

在适当的地方使用`instancetype`替代`id`可以提升代码的类型安全性。`instancetype` 只能用于方法的返回值类型。

一个OC的`property`只是用`@property`语法声明的方法。

`NS_ENUM`和`NS_OPTIONS`表达更简洁、更简单，同时声明了名称和类型，提升了代码完成度，并明确指明了数据类型和大小。兼容性也更好，老的编译器可以正确解析，新的编译器可以获知底层的类型信息。`NS_ENUM`的数据类型应该是`NSInteger`，`NS_OPTIONS`的数据类型应该是`NSUInteger`。

NS_ENUM`和`NS_OPTIONS`的命名和类一样，应该用驼峰命名法。

OC中，初始化方法分为`designated`和`convenience`，用`NS_DESIGNATED_INITIALIZER`来标记`designated`方法，其它为`convenience`方法，这种模式有利于在继承层次中确保初始化所有的实例变量。

`designated`初始化方法的限制：

- 必须调用父类的`designated`初始化方法。
- `convenience`初始化方法必须调用本类的`designated`初始化方法。
- 如果提供了一个或多个`designated`初始化方法，那么必须实现父类所有的`designated`初始化方法。

### 参考资源

- [Adopting Modern Objective-C](https://developer.apple.com/library/archive/releasenotes/ObjectiveC/ModernizationObjC/AdoptingModernObjective-C/AdoptingModernObjective-C.html)
- [禅与 Objective-C 编程艺术](https://github.com/oa414/objc-zen-book-cn)

## OC方法调用过程

编译器会把OC中的方法调用转化为一个（汇编）函数调用，可能的函数调用有：`objc_msgSend`、`objc_msgSend_stret`、`objc_msgSendSuper`和`objc_msgSendSuper_stret`。调用父类的方法会转化为`objc_msgSendSuper`，其它的会转化为`objc_msgSend`。返回值含有数据结构的会转化为`objc_msgSend_stret`和`objc_msgSendSuper_stret`。

函数签名如下：

```c
id _Nullable
objc_msgSend(id _Nullable self, SEL _Nonnull op, ...)
```

具体实现是汇编的。

`objc_msgSend`（就arm平台而言）步骤如下：

1. 判断receiver是否为nil
2. 从缓存里查找，找到了则分发，否则
3. 调用objc-class.mm中的`_class_lookupMethodAndLoadCache3`方法查找
   1. 如果支持GC，忽略掉非GC环境的方法（retain等）
   2. 从本class的method list寻找selector，如果找到，填充到缓存中，并返回selector，否则
   3. 寻找父类的method list，并依次往上寻找，直到找到selector，填充到缓存中，并返回selector，否则
   4. 调用`_class_resolveMethod`，如果可以动态resolve为一个selector，不缓存，方法返回，否则
   5. 转发这个selector，否则
4. 报错，抛出unrecognized selector异常

### 方法调用流程

![method_call_process](https://upload-images.jianshu.io/upload_images/4437917-fa0a35af378ac695.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/jpeg)

### 参考资源

* [iOS知识小集之为什么objc_msgSend()是用汇编实现的](https://zhongwuzw.github.io/2018/04/21/iOS%E7%9F%A5%E8%AF%86%E5%B0%8F%E9%9B%86%E4%B9%8B%E4%B8%BA%E4%BB%80%E4%B9%88objc-msgSend-%E6%98%AF%E7%94%A8%E6%B1%87%E7%BC%96%E5%AE%9E%E7%8E%B0%E7%9A%84/)
* [深入理解 Objective-C：方法缓存](https://tech.meituan.com/2015/08/12/deep-understanding-object-c-of-method-caching.html)
* [逐行剖析objc_msgSend汇编源码](https://www.jianshu.com/p/92d3fe62014d)

## 问题

**OC的类方法是否可继承？**

可以继承，OC中的类方法和实例方法本身的机制是一样的，`[self class]`本身也是一个对象。

**protocol是否可以定义类方法？**

可以，没什么区别。

**需要copy的setter方法中是否需要做类似`_name ！= name`的判断？**

不需要。对于需要copy的场景，如果copy的成本很高，做一些判断也无可厚非，但这种判断逻辑由外部来做更合理。这种情形并不常见，如果添加了判断，那么每次调用都会增加成本。

系统实现：

```objective-c
static inline void reallySetProperty(id self, SEL _cmd, id newValue, ptrdiff_t offset, bool atomic, bool copy, bool mutableCopy)
{
    if (offset == 0) {
        object_setClass(self, newValue);
        return;
    }

    id oldValue;
    id *slot = (id*) ((char*)self + offset);

    if (copy) {
        newValue = [newValue copyWithZone:nil];
    } else if (mutableCopy) {
        newValue = [newValue mutableCopyWithZone:nil];
    } else {
        if (*slot == newValue) return;
        newValue = objc_retain(newValue);
    }

    if (!atomic) {
        oldValue = *slot;
        *slot = newValue;
    } else {
        spinlock_t& slotlock = PropertyLocks[slot];
        slotlock.lock();
        oldValue = *slot;
        *slot = newValue;        
        slotlock.unlock();
    }

    objc_release(oldValue);
}
```

**如何理解`self`和`super`？**

`self`是类的隐藏参数，指向当前调用方法的这个类的实例。

`super`是一个Magic Keyword，它本质是一个编译器标识符，和`self`指向的是同一个消息接受者。`super`只是告诉编译器，调用方法时，直接去父类方法列表里面找，而不是先从本类开始查找。

假设有如下代码：

```objective-c
@implementation Son : Father
   - (id)init
   {
       self = [super init];
       if (self) {
           NSLog(@"%@", NSStringFromClass([self class]));
           NSLog(@"%@", NSStringFromClass([super class]));
       }
       return self;
   }
@end
```

输出结果都是`Son`。

编译器解析方法调用时，将`[super method]`转化为`objc_msgSendSuper`，而将其它的方法调用转化为`objc_msgSend`，返回值含有数据结构的会转化为`objc_msgSendSuper_stret`和`objc_msgSend_stret`。

**`init`方法中使用点语法有什么潜在问题？**

子类可能会覆写`setter`方法，

**什么情况下不会autosynthesis？**

1. 同时重写了readwrite属性的getter和setter方法。
2. 重写了readonly属性的getter方法。
3. 使用了`@dynamic`时
4. 在`@protocol`中定义的所有属性
5. 在category中定义的所有属性
6. 重载的属性。

**`objc_msgSend`为什么使用汇编实现？**

1. 我们无法定义一个`C`函数，可以有可变的参数(可变参数是可以实现的，参考`printf`函数)并且可以调用任意的`C`函数指针，因为函数指针类型是在是无穷无尽的，根本就无法预先全部定义出来。
2. 使用汇编另一个很重要的原因就是速度，首先，汇编就比`C`快，其次，通过使用汇编，可以免去大量局部变量拷贝的操作，参数会直接被存放在寄存器中，当找到`IMP`时，参数已经保存在了寄存器中，可以直接使用。

**`objc_msgSend`做了什么？**

1. 获取传递进来的类对象
2. 获取类用来缓存方法的cache
3. 使用`selector`在cache中查找
4. 如果cache中查找不到，则跳转到C代码(`_class_lookupMethodAndLoadCache3`)，进行slow search
5. 调用方法的IMP

**OC对象的内存结构？**

```c
/// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};

struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;

struct objc_ivar {
    char * _Nullable ivar_name                               OBJC2_UNAVAILABLE;
    char * _Nullable ivar_type                               OBJC2_UNAVAILABLE;
    int ivar_offset                                          OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
}
```

所有成员变量及父类的成员变量都存放在该对象所对应的存储空间中，编译器会预先计算好实例变量的偏移地址，访问实例变量时根据偏移地址来访问。

> 类似`objc_object`这样的结构体定义，内存空间是可扩展的，这是一种很常见的模式。

OC对象的内存结构如下：

![oc_object_structure](https://camo.githubusercontent.com/f446a82030de175ad18c7af85f097e1eea3fe496/687474703a2f2f692e696d6775722e636f6d2f376d4a6c556a312e706e67)

每个对象都有一个`isa`指针，指向它的类对象，类对象中存放着本对象的：

- 实例方法列表
- 成员变量列表
- 属性列表

类对象内部也有一个`isa`指针指向元对象（meta class），元对象内部存放的是类方法列表。

类对象内部还有一个`superclass`指针，指向它的父类对象。

![oc_object_isa_superclass](https://camo.githubusercontent.com/cdc02fffae7a70aa00cbb6c0f3675d00728cdaad/687474703a2f2f692e696d6775722e636f6d2f7736747a46787a2e706e67)

**`dealloc`流程？**

`dealloc` -> `object_dispose` -> `objc_destructInstance`。

```c
id 
object_dispose(id obj)
{
    if (!obj) return nil;

    objc_destructInstance(obj);    
    free(obj);

    return nil;
}

/***********************************************************************
* objc_destructInstance
* Destroys an instance without freeing memory. 
* Calls C++ destructors.
* Calls ARC ivar cleanup.
* Removes associative references.
* Returns `obj`. Does nothing if `obj` is nil.
**********************************************************************/
void *objc_destructInstance(id obj) 
{
    if (obj) {
        // Read all of the flags at once for performance.
        bool cxx = obj->hasCxxDtor();
        bool assoc = obj->hasAssociatedObjects();

        // This order is important.
        if (cxx) object_cxxDestruct(obj);
        if (assoc) _object_remove_assocations(obj);
        obj->clearDeallocating();
    }

    return obj;
}
```

解除所有的weak引用是在`clearDeallocating`中。

根据实验，`dealloc`方法开始时，所有的weak引用已经为nil了，不过断点调试或者用lldb命令调试的话，对象还不是nil，但程序的行为是nil。`unsafe_unretained`引用为非nil。具体原因还需要进一步探究。

**对象内存销毁时间表？**

1. 调用 -release ：引用计数变为零
     * 对象正在被销毁，生命周期即将结束.
     * 不能再有新的 __weak 弱引用， 否则将指向 nil.
     * 调用 [self dealloc] 
 2. 子类 调用 -dealloc
     * 继承关系中最底层的子类 在调用 -dealloc
     * 如果是 MRC 代码 则会手动释放实例变量们（iVars）
     * 继承关系中每一层的父类 都在调用 -dealloc
 3. NSObject 调 -dealloc
     * 只做一件事：调用 Objective-C runtime 中的 object_dispose() 方法
 4. 调用 object_dispose()
     * 为 C++ 的实例变量们（iVars）调用 destructors 
     * 为 ARC 状态下的 实例变量们（iVars） 调用 -release 
     * 解除所有使用 runtime Associate方法关联的对象
     * 解除所有 __weak 引用
     * 调用 free()

[参考链接](http://stackoverflow.com/a/10843510/3395008)

**IBOutlets应该用strong还是weak？**

都可以，但推荐用strong，weak主要用来避免循环引用，成本比strong要高。当outlet连接不是始终被view层次结构引用的subview或约束时尤其应该用strong。

**KVC/KVO可以操作非对象类型吗？**

可以，除对象类型，还支持NSNumber scalar类型以及NSValue结构类型。

## 常见错误

**clang -rewrite-objc报错**

```
cannot create __weak reference
      because the current deployment target does not support weak references
```

解决方案：

```
clang -rewrite-objc -fobjc-arc -stdlib=libc++ -mmacosx-version-min=10.7 -fobjc-runtime=macosx-10.7 -Wno-deprecated-declarations main.m
```

