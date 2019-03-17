# 浅论Swift与OC的区别

Swift作为最新的、现代化的编程语言，融合了一些最新的编程思想，在使用Swift时需要注意，如果用使用OC的方式使用Swift，那就是明珠暗投了。本篇谈一下Swift和OC的两个最核心的区别：Protocol和Value。

## Protocol

> [Protocols](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html) are used to define a “blueprint of methods, properties, and other requirements that suit a particular task or piece of functionality.”

我们都知道，OC是OOP（Object Oriented Programming）的编程语言，那么Swift呢？Swift是面向Protocol和Value的。

POP（Protocol Oriented Programming）解决什么问题呢？先看一个例子，假设有以下类层次结构：

![class_hierarchy](/Users/ypf/Desktop/class_hierarchy/class_hierarchy2.png)

如果需要D和F实现一个同样的功能，我们会怎么做呢？

某些程序员可能会直接将功能添加到A上，这样D和F就都可以继承这个功能了。但这么做存在一个问题，从功能特性的角度来讲，这个新功能很可能根本不属于A，长此以往下去，上层的类会增加很多不必要的功能。

当然我们也可以换一种更好的方案，将新功能封装在一个新类中，然后在D和F中包含这个新类。（composition over inheritance）

而在Swift中，我们可以将新功能定义为一个协议，让D和F实现这个协议。如果D和F的实现完全一样，我们可以为协议提供默认实现。这样就不会破坏继承结构，也不会导致上层类特性臃肿。

从Protocol的角度来讲，Swift和OC的一个主要区别就在于，Swift中可以给Protocol提供默认实现，以支持POP；另一个主要区别在于，Swift中不仅类可以实现Protocol，struct、enum等值类型也可以，因此Protocol的多态性比OC中纯继承的方式要强大很多。

## Value

> Local Reasoning — Self contained thinking about functions. You shouldn’t have to understand the whole context of the application to maintain individual functions. You should be able to just apply local reasoning to understand individual functions.

Swift和OC的另一个主要区别就是，Swift中的struct、enum等值类型的功能比OC中强大很多，可以有方法，可以实现Protocol。

值类型和引用类型最重要的区别在于，值类型是不可变的。Immutability是最新的编程思想着重强调的一个特性，它带来了很多好处，比如更利于Local Reasoning和多线程同步。

Local reasoning是一种通用的技术，并不特定于UI编程、Swift或任何Swift概念。它是一种全新的编程思维方式，在你将来所有的设计中都应该考虑它。值类型是一个支持代码Local Reasoning的非常重要的方面。



## 参考资源

* [*Protocol-Oriented Programming in Swift*](https://developer.apple.com/videos/play/wwdc2015/408/)