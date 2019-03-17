# 如何写一个最糟糕的应用？

根据我的经验，要想提升自己对设计模式、架构的认知水平，最快的方法就是重构烂代码。要想增进对最新编程思想、技术的理解，最快的方法就是重构烂代码。理由很简单，这些东西都是为了应对烂代码而生的，所谓烦恼即菩提，就是这个意思。

当我看到大神Matt Gallagher的[The worst possible application](https://www.cocoawithlove.com/blog/worst-possible-application.html)这篇文章时，不由得会心一笑。不打算全篇翻译，此篇算作解读版吧，感兴趣的建议读原文，以下为文章内容。

在本文中，我打算写一个最糟糕的应用。怎么做到呢？当你看到一段糟糕无比的代码时，有没有产生过这个疑问：这个家伙是怎么做到的？写的这么烂真是太不容易了。此应用会违背最主要的一个应用设计原则：model和view分离。

目的是什么呢？试图说清楚一个问题：一个设计模式在代码层面的直接成效是什么？在一个原本简单、清晰的实现基础上，如果采用最糟糕的设计模式，应用会产生哪些问题？如果换成一个更加通用的设计模式，问题能否得到显著改善？

## 充满矛盾的话题

当讨论应用的设计模式时，想要做出精确而明白的陈述是非常困难的一件事情，因为在含糊的、高层次的陈述和具体的实现之间存在巨大的鸿沟，以至于实现可能根本无法反映设计意图。

出于无奈，我决定反其道而行，用最糟糕的设计模式写一个应用。或许，当我犯了很多严重错误之后，设计模式会显现出激动人心的显著效果，然后我就可以清晰、明确地展开讨论了。

## 最糟糕的模式

为了找出最糟糕的设计模式，我设计了5种自认为严重违背MVC模式的情景，并跟4位同事做了讨论，就像我真的在实际项目中遇到的问题一样。

情景如下：

```
一个4500行代码的UIViewController，修改任何方法都可能会影响到每个view。
```

有人指出，这是代码实现的问题，而不是设计模式的问题，通过标准的重构和分解即可解决这个问题。

```
Model包含CoreGraphics绘图代码，并通过闭包传递给View。
```

一些人认为这是一种合理的方案，或许是因为它看起来跟传递SVG数据差别并不大吧。我依然认为它有潜力成为最糟糕的方案之一。

```
Model持有对Controller的引用，当数据发生变更时，Model直接调用Controller的刷新方法。
```

这可能是我提出的最愚蠢的情景了，主要是因为解决方案实在太简单了（使用合理的监听或通知）。

```
有三个独立的Model，View Controller负责Model的同步。
```

如果你跟别人说，你需要在一个View Controller中同步三个来源的数据，这听起来不过是正常的业务场景，人们会给出各种建议，这可不够糟糕。

我希望找到一种能够满足以下条件的架构错误：

- 没有人说，这个问题可以通过简单的重构来解决
- 没有人认为应该这样开发app

继续努力...

```
没有独立的Model。数据直接保存在View中，Controller负责同步更新以及View之间的通信。
```

这种错误其实并没有什么大惊小怪的（就是大名鼎鼎的massive view controller），不过Model-View-Controller为大众所熟知已经有几十年了，“Model和View不分离”可以被认为是最糟糕的错误设计。

> 解决者：这种错误设计在初级iOS开发者中可以说司空见惯，几乎是最为原始的一种App实现方式。有兴趣的可以看一下Massive View Controller相关的主题。
>
> 在这种最原始的设计之上展开讨论，我想再合适不过了，哪怕初学者也可以从中受益。

终于找到了，我们来实现一个看看。

## 不分离

“Model和View不分离”到底指什么呢？

很不幸，和其它设计模式问题一样，很难给出一个具体、明确的解释。无数卓越的作者对这个主题作过论述（[分离主题的文章](https://martinfowler.com/eaaDev/SeparatedPresentation.html)），但是依然没有一个清楚的规则。

我打算对**Model和View分离**使用以下规则：

1. Model接口必须纯洁地封装所有的应用状态，不引用任何View或应用框架。
2. 非Model组件可以通过Model的接口发起更新，但是只能请求执行某个动作，而不能直接操作原始数据。
3. 请求Model执行某个动作之后，非Model组件不能立即更新或刷新任何非Model状态。依赖于Model的更新只能来源于Model的变更通知。

然后，“Model和View不分离”可以定义为：违反了一条或多条以上规则。

可能一下子无法弄明白选择这些规则的理由，简而言之：强制将Model与应用的其它部分隔离，禁止其它部分假设Model的内部实现。

```
Spoilers: 后面会讨论“抽象泄露”。非Model组件应该小心地避免假设Model的内部实现为什么如此重要？避免抽象泄露是真正的原因。
抽象泄露：软件开发时，本应隐藏实现细节的抽象化不可避免地暴露出底层细节与局限性。抽象泄露是棘手的问题，因为抽象化本来目的就是向用户隐藏不必要公开的细节。
抽象泄露法则：所有重大的抽象机制在某种程度上都是有漏洞的。
```

## 一个展示应用

我发现自己之前分享的一个应用就是没有分离Model和View的。几个月之前，我[分享了自己将近20年前的“Mines”代码](https://www.cocoawithlove.com/blog/porting-from-macos8-to-sierra.html)。由于使用的是90年代古典的MacOS C++，不便于讨论，因此我花了几个小时用Swift做了重新实现。

最终成果，完全没有Model分离，[Mines for iOS](https://github.com/mattgallagher/CwlWorstPossibleApplication):

![Mines_for_iOS](https://www.cocoawithlove.com/assets/blog/mines_screenshot.png)

<center><i>Mines for iOS</i></center>

## 应用快速总结

应用的内容是雷区。我创建了100个方块（放在10*10的方格中），15个地雷随机分布在这些方块中，应用开始时每个方块都是被掩盖的，之后可以被标记或揭开。

此应用不需要其它数据了，不过我另外缓存了每个方块的相邻地雷数量以及获胜之前需要揭开的不包含地雷的方块数量。此应用还有一个“Flag mode”的开关，不过它是一个“暂时的view-state”，被从Model中排除掉了。

如果此应用有独立的Model，那么应该包括：

* 100个方块，每个方块包含`isMine`、`adjacent`和`covered`属性
* `nonMineSquaresRemaining`的数量
* 生成初始方块的函数，包括`isMine`的分布和`adjacent`的数量
* 点击方块的处理函数，标记或揭开方块，以及更新`nonMineSquaresRemaining`

当然，此应用还没有明智到拥有一个独立的Model。

## 打破所有规则

让我们比较一下之前分离Model的规则，看看我做了哪些糟糕的事情。

### 1. Model应该是密封的，并且没有对View或应用框架的引用

表示应用中方块的`SquareView`对象（UIButton的子类）包含以下属性：

```swift
var covering: Covering = .covered
var isMine: Bool = false
var adjacent: Int8 = 0
```

这并不是对保存在其它地方的数据的展示，而且对雷区的*唯一*表示，雷区的状态分散在100个按钮当中。

完全违背了规则1。

### Model的接口应该暴露action，而不是对原始数据的操作

游戏中最主要的更改发生在方块被点击的时候，怎么做的呢？

```
// In a function on the GameViewController...
squareViews[index].covering = .uncovered
```

简言之，当`SquareView`被点击时，它发送事件给`GameViewController`，`GameViewController`维护了组成雷区的`SquareView`数组。`GameViewController`直接查找到被点击的`SquareView`，然后直接修改它的`covering`属性。

完全违背了规则2。

### 更新仅发生在对Model变更通知的响应

当点击一个地雷时的代码如下：

```swift
if squareView.isMine {
   squareView.covering = .uncovered
   nonMineSquaresRemaining = -1
   refreshSquaresToClear()
   squareView.setNeedsDisplay()
   return
}
```

当`GameViewController`改变`SquareView`的`covering`属性或自己的`nonMineSquaresRemaining`时，必须同时调用`SquareView`的`setNeedsUpdate`方法和自己的`refreshSquaresToClear`方法以刷新界面。

完全违背了规则3。

## 结果有多糟糕？

`GameViewController`就像两个完全不同的类组合起来的一样 - 很明显它做的事情太多了。

`SquareView`（UIButton的子类）包含`init(fromDictionary:)`和`toDictionary()`方法，以便当`UIStateRestoration`时实现序列化和反序列化 - 可笑的职责错配。

不过此应用并不是一团糟，底层逻辑依然清晰而简单，没有什么地方看起来费力或混乱。

我想要一个恐怖片，但这个并没有什么特别可怕的，或许实验不奏效吧。

## 恰当分离的Model-View-Controller

或许我需要跟一个更加典型的Model-View-Controller实现做直接对比以突显出不当之处。

具体实现可以看[仓库中的“separated”分支](https://github.com/mattgallagher/CwlWorstPossibleApplication/tree/separated)。

此版本中，在新建的`Game.swift`文件中增加了`Square`和`Game`两个类型。`Game`是Model的接口，它唯一的变更方法是`tapSquare`，变更只能监听其`Game.changed`通知。

`Square`包含了之前`SquareView`中游戏相关的数据，而`Game`包含了之前`GameViewController`中游戏相关的数据：

```swift
struct Square: Codable {
   var covering: Covering = .covered
   var isMine: Bool = false
   var adjacent: Int8 = 0
}

class Game: Codable {
   private(set) var squares: Array<Square>
   private(set) var nonMineSquaresRemaining: Int
}
```

`SquareView`依然包含自身绘制需要的数据，不过此时的数据仅仅是`Game`中相应的`Square`对象的拷贝而已。

```swift
class SquareView: UIButton {
   var square: Square { didSet { setNeedsDisplay() } }
}
```

除了移动数据成员位置之外，`GameViewController`中的以下方法也被移到了`Game`类中：

```swift
func loadGame(newSquareViews: Array<SquareView>, remaining: Int)
func newMineField(mineCount: Int) -> Array<SquareView> {
func uncover(squareViews: Array<SquareView>, index: Int) -> Int {
func iterateAdjacent(squareViews: Array<SquareView>, index n: Int,
   process: (Array<SquareView>, Int) -> ()) {
```

`GameViewController`中的按钮点击事件处理方法：

```swift
@objc func squareTapped(_ sender: Any?)
```

其内容也被移到了`Game`类的`tapSquare`方法中（`GameViewController`的`squareTapped`方法调用`Game`的`tapSquare`方法）。

由于`SquareView`已经不包含主要的表示数据了，因此`toDictionary()`和`init(fromDictionary:)`方法也就不需要了。`Game`类遵循`Codable`协议，因此Model自动支持序列化。

最后，`GameViewController`监听`Game`的变更通知以更新view：

```swift
NotificationCenter.default.addObserver(self, selector: #selector(gameChanged(_:)),
   name: Game.changed, object: newGame)
```

## 比较揭示了什么？

有一些小的功能变化，但**最大的变化是代码的移动**。

初看起来并没有什么特别 - 修复应用程序设计中最严重的错误居然只是移动了一些代码而已。难道代码组织不重要吗？

### 好吧，我说谎了

在我说明代码组织为什么那么重要之前，我必须更诚实一些。其实“移动一些代码”只是表面，还涉及更多其它工作。尤其是：

- 辨识出了Model数据
- 确定了纯粹作用于Model数据的所有函数
- 创建了一个通知系统以输出Model数据
- 确保函数和通知提供了一个与Model交互的简单方式
- 围绕整个概念实现了一个接口

通过并行分析很难发现所有这些工作，因为**我已经在没有分离的版本中完成了大部分工作**。分离Model的观念早已经深植于我头脑之中，以至于我在试图避免它时也创建了一个事实上的Model。

为了突显出这种天生的分离，我重排了`GameViewController`中函数的顺序。看一下[Mines未分离版本中的GameViewController](https://github.com/mattgallagher/CwlWorstPossibleApplication/blob/master/CwlWorstPossibleApplication/GameViewController.swift)，你会发现`GameViewController`中101行以上的代码正好与Model中代码相对应。这些代码虽然没有被拆分到另一个类中，但也没有直接和View相关的代码混在一起。

即使在写这篇文章的时候，当我给出“应用快速总结”时，我也花了所有的时间来描述*Model*，即使这个应用程序当时在技术上还没有Model。

是的，代码比较主要体现在了代码的移动，但这只是让代码组织反映出我已经写过的东西 - 对游戏数据和逻辑事实上的抽象（即使被混杂和遮蔽）。

### Model是一种抽象

分离版的Model应用实现澄清了Model的抽象并强制其执行。

通过抽象，`Game`接口内部执行了很多隐藏工作，但从外部看起来却非常简单。它暴露了两个非私有方法 - `init`和`tapSquare` - 它们对应游戏过程中用户仅有的两个操作（开始游戏和点击方块）。游戏的输出即雷区的状态通过`Game.changed`通知来传达。

`Game`的抽象将100多行的游戏创建和维护代码简化为了两个动作和一个输出。尽管额外需要一些监听的代码（在`GameViewController`中，差不多10行代码），从`GameViewController`的角度来看，这种抽象依然明显减小了代码大小和复杂度。

这就是应用设计模式中Model的重要作用，它让你减小了复杂度，通过：

- 减少应用其它部分所需的步骤
- 消除了应用其它部分确保正确性的需要
- 操作位置不需要知道状态监听的位置

让Model可以自由选择是非常重要的，因为Controller和View无法自由选择。（译者：此处的自由选择主要指没有平台依赖）`UIViewController`和`UIView`的功能主要由UIKIt框架的实现约束和要求来决定的，因此无法随意抽象。Model是非常有价值的，因为它不依赖应用程序框架。

### 有漏的和不好的抽象

如前所说，在正式分离Model之前，已经使用了一个事实上的Model抽象。然而，抽象只有在采取步骤避免外部组件假设其内部工作时才能降低复杂性——事实上的抽象并没有对其实现提供真正的隐藏或屏蔽。

当外部组件假设抽象的内部工作时，我们称此抽象为“有漏的”抽象。如果接口没有隐藏工作，那么它根本不算抽象。

例如，假设我分离Model的时候，没有把所有的函数移动`Game`结构体中，而仅仅移动了数据成员并将它们作为可变的属性暴露出来：

```swift
class Game: Codable {
   var squares: Array<Square>
   var nonMineSquaresRemaining: Int
}
```

如果这就是完整的Model定义，所有的维护工作依然在`GameViewController`中，如之前一样。没有抽象的Model接口并没有给程序带来任何好处。

### 应用设计模式可能无用

这就是很难毫不含糊地讨论应用设计模式的原因。Model*应该*是抽象的，*应该*降低复杂度和提高代码和逻辑的局部性。但是否真能如此还取决于你的实现。如果Model的抽象选择不当，可能分离根本带不来好处。

应用设计模式并不提供实现，而只是建议使用一些组件 - 这些组件本身是完全无用的。尽可能为这些组件使用好的抽象是程序员的责任。遵循良好的应用设计模式*应该*会产生良好的抽象，但这并不会自然发生。

思考应用设计模式的一种有用方式是把它看作一种思维实验，它要求我们从不同的角度来解释我们的程序。它向我们提出了这个问题：如果你被迫通过这些组件之间的接口来路由程序中的更新或操作，在边界处你将如何解释程序的状态和行为？这个问题鼓励在特定的有利位置抽象需求和可用选项。通过此视角考虑问题是否能让程序行为的抽象清晰而简单，这将决定该应用设计模式是否对程序有益。

## 总结

根据你对Model的抽象，可能出现以下两种情况：

1. 将Model从View中分离基本上只是挪动了一下代码的位置。
2. 通过仔细的抽象、更好的分离关注点、更好的代码局部性和更加简单的数据依赖传递，将Model和View分离可以降低整体实现的复杂度。

事实上，以上描述可能适应于同一个应用，就像[Mines app](https://github.com/mattgallagher/CwlWorstPossibleApplication)一样。

编写没有分离Model的应用程序被认为是最糟糕的应用程序设计，这并不是因为它会立即导致灾难，而是因为它表明你从未尝试明确地隔离程序的功能。我们编写应用程序不是因为我们喜欢构造视图和控制器——它们只是达到目的的一种手段。我们编写应用程序来展示我们的Model，对其执行操作并查看结果。

最糟糕的应用设计是不理解应用*是什么*，进而随意散布代码而模糊其本质。

> 解读：随意堆砌代码并不是什么稀奇的事情，对于很多人来说，只要代码能跑起来，测试没有明显的bug，就已经不错了。至于设计是否合理，代码是否高效，根本不在考虑范围内。当然，对于很多人来说，这也是无奈之举，只是不要满足于原始状态，向更高等级演化吧。

## 前瞻

在Model-View-Controller的典型描述中，Model包含应用的所有数据，而View只是数据的展示。然而，经常有一些影响View显示的数据，如滚动状态、导航状态、未提交的输入数据等，这些数据仅存在于View中，而不是Model中，这显然违背了Model-View-Controller原则。

下一篇文章，我会讨论一下，当我们要求view-state有自己的Model时会发生什么，以及view-state如何驱动整个应用。

## 快速笔记

毫不含糊地对应用设计模式做准确无误地论述是非常困难的，因为在模糊的、高层次的设计和特定实现之间距离比较远。

MVC设计模式早已深入人心，Model和View分离在绝大多数人看来已经是理所当然的。

良好的设计模式并没有指明具体如何实现，因此特定的实现可能根本没有达到设计的目的。

Model应该是抽象的，隐藏其内部实现细节，外部不应该假设其内部实现。