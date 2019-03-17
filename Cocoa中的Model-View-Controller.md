# Cocoa中的Model-View-Controller

根据苹果的文档，Cocoa应用程序的标准模式被称为Model-View-Controller。尽管同名，但此模式与应用于Smalltalk-76的Model-View-Controller的原始定义却非常不同。Cocoa的应用设计模式实际上与Taligent（一个苹果公司在上世纪90年代合作开发的项目）开发的想法有更多的共同点，而不是Smlltalk这个词源。

此文中，我们看一下Cocoa中使用的最主要应用设计模式背后的一些理论和历史。我将讨论Cocoa的Model-View-Controller方式的关键缺陷，以及苹果为解决这一缺陷而放弃的努力，并想知道下一个主要改进将来自何处。

## Smalltalk-76

> 或许UI开发领域被引用最广泛的模式就是Model View Controller（MVC）了 - 它也是被错误引用最多的。我已经记不清楚多少次看到MVC被描述成完全不同的东西。- Martin Fowler，[GUI Architectures](https://www.martinfowler.com/eaaDev/uiArchs.html)

我想先快速解释一下上面引用中Martin Fowler的意思，根据Fowler使用的定义（这个定义最初由Trygve Reenskaug用来描述Smalltalk-76），Cocoa开发中广泛使用的并不是Model-View-Controller。

在Smalltalk-76中，交互视图被分为两个完全独立的对象：View对象和Controller对象。View对象执行显示，但任何的点击或交互都不是由View对象处理的，而是由合作的Controller对象分派的。理解的关键点是，Controller不加载、设置或管理View，也没有一个Controller处理多个View的操作。在Model-View-Controller的原始定义中，View和Controller只是用于屏幕上单个控件的显示和处理。

![Smalltalk_MVC](https://www.cocoawithlove.com/assets/blog/smalltalk_mvc.svg)

<center><i>Smalltalk-76's version of Model-View-Controller</i></center>

从Smalltalk的Model-View-Controller示意图可以看出，*Model*是对象图的核心组件，通信主要直接发生在Model和View或Controller之间。

这种准确的模式反映了Smalltalk-76如何处理用户输入，在现代程序中几乎不需要使用这种精确模式。从此意义上说，要么没有现代框架是真正的Model-View-Controller，要么这个术语的定义已经被用来代表其它含义。

## Cocoa (AppKit/UIKit)

Cocoa中说到Model-View-Controller时，主要是试图引出应用设计中[分离展示和内容](https://en.wikipedia.org/wiki/Separation_of_presentation_and_content)的概念（Model和View的设计应该是解耦的，并且在构造时被松散连接在一起）。公平地说，不只是Cocoa以这种方式使用Model-View-Controller：这个术语的大多数现代用法实际上是为了表达Martin Fowler所称的“分离表示”，而不是最初的Smalltalk-76定义。

看一下Cocoa实际使用的精确模型，Apple的Cocoa参考指南使用的Model-View-Controller定义是这样的：

![Cocoa_MVC](https://www.cocoawithlove.com/assets/blog/cocoa_mvc.svg)

<center><i>Cocoa's version of Model-View-Controller</i></center>

需要注意的重要一点是，Controller是对象图的中心，大多数的通信都是通过Controller进行的 - 与Smalltalk-76版的不同，即Model是图的中心。

Cocoa并不强制app必须使用这种模式，但所有的应用程序模板都强烈地暗示了这一点。从NIB文件加载强烈鼓励了NSWindowController/UIViewController的使用。NSTableView/UITableView的delegate需求和其它相关类强烈暗示了一个理解整个表示职责的协调器类。像UITabBarController和UINavigationController这样的类显式地要求UIViewController的实例以协调View。

## Taligent

在学术讨论中，Cocoa所称的Model-View-Controller模式通常被称为[Model-View-Presenter](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter)。两者是一样的，除了Cocoa中所称的Controller被称为“Presenter”。“Presenter”这个名字