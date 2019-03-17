# iOS事件处理

## 事件

事件指什么呢？看一下事件类型的定义：

```objc
typedef NS_ENUM(NSInteger, UIEventType) {
    UIEventTypeTouches, // 触摸事件
    UIEventTypeMotion, // 动作事件，目前主要是摇动
    UIEventTypeRemoteControl, // 远程控制事件，如音乐的播放、暂停、停止等
    UIEventTypePresses NS_ENUM_AVAILABLE_IOS(9_0), // 按压事件
};
```

具体的事件子类型：

```objective-c
typedef NS_ENUM(NSInteger, UIEventSubtype) {
    // available in iPhone OS 3.0
    UIEventSubtypeNone                              = 0,
    
    // for UIEventTypeMotion, available in iPhone OS 3.0
    UIEventSubtypeMotionShake                       = 1,
    
    // for UIEventTypeRemoteControl, available in iOS 4.0
    UIEventSubtypeRemoteControlPlay                 = 100,
    UIEventSubtypeRemoteControlPause                = 101,
    UIEventSubtypeRemoteControlStop                 = 102,
    UIEventSubtypeRemoteControlTogglePlayPause      = 103,
    UIEventSubtypeRemoteControlNextTrack            = 104,
    UIEventSubtypeRemoteControlPreviousTrack        = 105,
    UIEventSubtypeRemoteControlBeginSeekingBackward = 106,
    UIEventSubtypeRemoteControlEndSeekingBackward   = 107,
    UIEventSubtypeRemoteControlBeginSeekingForward  = 108,
    UIEventSubtypeRemoteControlEndSeekingForward    = 109,
};
```

> 除了以上类型的事件之外，`responder`对象还需要处理Editing Menu消息。对应API中的如下方法和属性：

```objective-c
- (BOOL)canPerformAction:(SEL)action withSender:(nullable id)sender NS_AVAILABLE_IOS(3_0);
// Allows an action to be forwarded to another target. By default checks -canPerformAction:withSender: to either return self, or go up the responder chain.
- (nullable id)targetForAction:(SEL)action withSender:(nullable id)sender NS_AVAILABLE_IOS(7_0);

@property(nullable, nonatomic,readonly) NSUndoManager *undoManager NS_AVAILABLE_IOS(3_0);
```

## 使用响应者和响应者链处理事件

如何处理应用内传播的一系列事件？

### 概述

应用通过`responder`对象接受和处理事件。`responder`对象是[`UIResponder`](https://developer.apple.com/documentation/uikit/uiresponder)类的实例，常见的子类包括[`UIView`](https://developer.apple.com/documentation/uikit/uiview)、[`UIViewController`](https://developer.apple.com/documentation/uikit/uiviewcontroller)和[`UIApplication`](https://developer.apple.com/documentation/uikit/uiapplication)。`responder`接受原始的事件信息，然后要么自己处理事件或者将事件转发给下一个`responder`。当应用接受到一个事件之后，UIKit会自动将事件交给最适合的`responder`处理，称之为`first responder`。

没有被处理的事件会沿着`responder chain`传递，`responder chain`由应用的`responder`对象动态配置而成。下图展示的是一个包含标签、输入框、按钮和两个背景View的界面的`responder`对象，图中同时展示了事件如何沿着`responder chain`传递。

![responder_chain_demo](https://docs-assets.developer.apple.com/published/7c21d852b9/f17df5bc-d80b-4e17-81cf-4277b1e0f6e4.png)

> 注意：AppDelegate不一定是`responder`，尽管创建应用时XCode生成的模板代码默认将AppDelegate继承于`UIResponder`。
>
> 可以动手试一下，将AppDelegate的父类改为`NSObject`，看一下应用是否可以正常运行？理论上来说，只要没有在AppDelegate中做事件处理，那么就没有影响。

### 谁是`First Responder`？

UIKit根据事件的类型来确定`first responder`：

| Event type            | First responder      |
| --------------------- | -------------------- |
| Touch events          | 当前触摸的View       |
| Press events          | 当前获得了焦点的对象 |
| Shake-motion events   | 创建时指定的对象     |
| Remote-control events | 创建时指定的对象     |
| Editing menu messages | 创建时指定的对象     |

> 加速器、陀螺仪、磁力计相关的motion事件并不沿着responder chain传递，Core Motion会把事件直接交给创建时指定的对象。

UIControl通过发送action消息直接与target对象通讯(target-action机制)，尽管action消息不属于事件，但依然可以通过`responder chain`传递。当target对象为nil时，UIKit会沿着`responder chain`寻找实现了相应action方法的对象。例如，editing menu就是采用这种模式寻找类似[`cut(_:)`](https://developer.apple.com/documentation/uikit/uiresponderstandardeditactions/2354193-cut)、[`copy(_:)`](https://developer.apple.com/documentation/uikit/uiresponderstandardeditactions/2354191-copy)或[`paste(_:)`](https://developer.apple.com/documentation/uikit/uiresponderstandardeditactions/2354189-paste)方法。

Gesture recognizer先于View接受触摸和按压事件，当手势识别器没有识别触摸动作时，UIKit会将事件交给View，然后沿`responder chain`传递。

### Touch事件属于谁？

UIKit使用hit-testing来确定touch事件的发生位置，默认实现是比较touch的位置和view层次结构中view的bound。UIView的[`hitTest(_:with:)`](https://developer.apple.com/documentation/uikit/uiview/1622469-hittest)方法遍历view层次结构，寻找最深次的bound包含touch的子view，这个子View将成为touch事件的第一响应者。

> 当touch位于一个View的bound之外时，[`hitTest(_:with:)`](https://developer.apple.com/documentation/uikit/uiview/1622469-hittest)方法会忽略这个View以及其所有的子View。因此，当view的[`clipsToBounds`](https://developer.apple.com/documentation/uikit/uiview/1622415-clipstobounds)属性是false时，超出bound之外的子view默认是不接受事件处理的。

当touch事件发生时，UIKit会创建一个[`UITouch`](https://developer.apple.com/documentation/uikit/uitouch)对象并与view关联。当touch的位置或其它参数改变时，UIKit会更新UITouch对象。唯一不会改变的属性是view，即使touch的位置已经移出了view。当touch结束时，UIKit会释放掉UITouch对象。

### 改变响应者链

响应者链并不是固定不变的，可以通过重写`nextResponder`属性来更改。默认的响应者链就是UIKit对象通过这种方式来实现的。

### 问题

**iOS中有哪些事件？**

触摸事件；摇动等动作事件；远程控制事件；按压事件；

**iOS的事件处理机制？**

iOS中，由响应者对象负责接受和处理事件，常见的响应者对象包括UIView、UIViewController、UIWindow、UIApplication等。当响应者对象接受到事件但无法处理时，会将事件交给下一个响应者，响应者对象依次链接形成一个链，称为响应者链。当app接受到事件时，会根据相应的情形确定第一个响应者对象，这个响应者就被称为第一响应者。

**什么是响应者链？**

响应者链是由响应者对象通过`nextResponder`属性关联起来的一系列用来接受和处理应用事件消息的对象。

**Target-Action和响应者链的关系？**

Target-Action是一种用来处理用户事件的典型模式，对事件感兴趣的对象(Target)可以向控件注册，当相应的事件发生时，控件会发送Action消息给已注册的对象，是一种极为松散耦合的模式。

当target对象为nil时，action消息会沿响应者链传递。