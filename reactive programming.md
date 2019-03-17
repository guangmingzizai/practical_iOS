# reactive programming

Reactive programming addresses the following problem:

```
Any “getter” for mutable state causes problems because it gives you the current value without ensuring you respond to the changes.
```

by suggesting the following solution:

```
Never store mutable state on your types. Instead, when you generate a new value in response to a change, send the value into a channel. Any part of the program that relies upon that value must subscribe to the channel.
```

Simply put: reactive programming manages asynchronous data flows between sources of data and components that need to *react* to that data.

With reactive programming, all of the threading and lifetime management that caused a huge burden for the previous implementation are implicit – they’re still *there* but they’re just handled internally.

Reactive programming changes how data is stored, how it flows through your program and how the elements of your program are connected. The result is significant improvement across the following categories:

- Thread safety
- Coordinating concurrent asynchronous tasks
- Loose coupling of components
- Data dependencies

The biggest advantage comes when you realize that in applying a solution to just one of these problems, you’ve gained a solution to the other three for free.



## FRP

Functional Reactive Programming (FRP) is now 20 years old. Although originally motivated by interactive 3D computer graphics, FRP is a general paradigm for describing dynamic (time-varying) information. Such information had traditionally been described in software only indirectly, as successive side effects of sequential execution. In contrast, FRP expressions describe entire evolutions of values over time, representing these evolutions directly as first-class values. From the start, FRP has been based on two simple and fundamental principles, namely (a) having a precise and simple denotation and (b) continuous time. The first property, which Peter Landin called "denotative" (and "genuinely functional"), applies across problem domains and ensures a precise, implementation-independent specification, insulated from operational details as found in efficient implementations. As such, denotative systems can be reasoned about practically and rigorously. The second property (temporal continuity) is domain-specific and is crucial for simple composability, natural specification of behavior via integration and differentiation, and adaptively efficient implementations.

Over the last few years, something about FRP has generated a lot of interest among programmers, inspiring several so-called "FRP" systems implemented in various programming languages. Most of these systems, however, lack both of FRP's fundamental properties. Missing a denotation, they're defined only in vague and/or operational terms (e.g. "graphs" and "update propagation"). Missing continuous time, they fail to provide temporal modularity (sampling-independence and natural temporal transformability), committing prematurely to sampling rates that may turn out to be too low for accuracy or too high for efficiency. For the same reason, these systems cannot express behaviors as integrals or derivatives and must instead express explicit approximations, leading to cluttered code with poor quality and/or performance. (Discrete notions of imagery have these same drawbacks, remedied by vector graphics and other continuous models.)

### FRP’s two fundamental properties

- Continuous time. (Natural & composable.)
- Precise, simple denotation. (Elegant & rigorous.)

Deterministic, continuous “concurrency”.

Warning: most modern “FRP” systems have neither property.

## 关注点

- 线程
- 生命周期



## 参考资源

- [operators](http://reactivex.io/documentation/operators.html)

- [Reactive Programming versus Reactive Systems](https://www.lightbend.com/reactive-programming-versus-reactive-systems)

- [The Essence of FRP](https://begriffs.com/posts/2015-07-22-essence-of-frp.html)

- [The essence and origins of FRP](http://conal.net/talks/essence-and-origins-of-frp-bayhac-2015.pdf)

  



## 问答

### reactive programming和FRP（functional reactive programming）的区别？

平时所说的reactive programming，完整的说法应该是imperative reactive programming。

### reactive programming解决了什么问题？如何解决的？



