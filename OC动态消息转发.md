# OC动态消息转发

## 消息转发流程：

![message_forward](https://upload-images.jianshu.io/upload_images/1395150-aa440fea34187cd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/588)

总体原则：**先自己，后转发；先单播，后多播；**

## 方法介绍

### 动态给类添加方法实现

```objc
/**
* Dynamically provides an implementation for a given selector for a class method.
*/
+ (BOOL)resolveClassMethod:(SEL)sel;
/**
* Dynamically provides an implementation for a given selector for an instance method.
*/
+ (BOOL)resolveInstanceMethod:(SEL)sel;
```

这两个方法允许动态地为给定selector提供实现。

一个OC方法只不过是最少接受两个参数——self和_cmd的C函数。通过`class_addMethod`函数，可以给类添加一个作为方法的函数。给定以下函数：

```c
void dynamicMethodIMP(id self, SEL _cmd)
{
    // implementation ....
}
```

你可以通过`resolveInstanceMethod:`动态地将它添加为类的方法（方法名为resolveThisMethodDynamically）：

```objc
+ (BOOL)resolveInstanceMethod:(SEL)aSEL
{
    if (aSEL == @selector(resolveThisMethodDynamically))
    {
          class_addMethod([self class], aSEL, (IMP) dynamicMethodIMP, "v@:");
          return YES;
    }
    return [super resolveInstanceMethod:aSel];
}
```

### 将消息转发给其它对象

```objc
// Returns the object to which unrecognized messages should first be directed.
- (id)forwardingTargetForSelector:(SEL)aSelector;
```

如果一个对象实现了此方法，并返回了一个non-nil的值，那么消息将转发给这个对象。很显然，如果返回`self`，那么将进入死循环。

此方法允许我们在进行更加昂贵的`forwardInvocation:`机制之前将消息转发给另一个对象。当我们需要在消息转发过程中截获invocation、修改参数或返回值时，此方法并不适用。

### 

## 如何实现一个MultiDelegate

需要将方法调用转发给多个对象，因此只能用最灵活的方式：

```objective-c
- (BOOL)respondsToSelector:(SEL)selector {
    if ([super respondsToSelector:selector])
        return YES;
    
    for (id delegate in _delegates) {
        if (delegate && [delegate respondsToSelector:selector])
            return YES;
    }
    
    return NO;
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)selector {
    NSMethodSignature* signature = [super methodSignatureForSelector:selector];
    if (signature)
        return signature;
    
    [_delegates compact];
    if (self.silentWhenEmpty && _delegates.count == 0) {
        // return any method signature, it doesn't really matter
        return [self methodSignatureForSelector:@selector(description)];
    }
    
    for (id delegate in _delegates) {
        if (!delegate)
            continue;

        signature = [delegate methodSignatureForSelector:selector];
        if (signature)
            break;
    }
    
    return signature;
}

- (void)forwardInvocation:(NSInvocation *)invocation {
    SEL selector = [invocation selector];
    BOOL responded = NO;
    
    for (id delegate in _delegates) {
        if (delegate && [delegate respondsToSelector:selector]) {
            [invocation invokeWithTarget:delegate];
            responded = YES;
        }
    }
    
    if (!responded && !self.silentWhenEmpty)
        [self doesNotRecognizeSelector:selector];
}
```



## 如何实现一个WeakProxy?

功能描述：弱引用一个对象，当对象存在时，将消息转发给这个对象，当对象被释放时，不触发*`doesNotRecognizeSelector:`*。

```objc
#pragma mark Life Cycle

// This is the designated creation method of an `FLWeakProxy` and
// as a subclass of `NSProxy` it doesn't respond to or need `-init`.
+ (instancetype)weakProxyForObject:(id)targetObject
{
    FLWeakProxy *weakProxy = [FLWeakProxy alloc];
    weakProxy.target = targetObject;
    return weakProxy;
}

#pragma mark Forwarding Messages

- (id)forwardingTargetForSelector:(SEL)selector
{
    // Keep it lightweight: access the ivar directly
    return _target;
}

#pragma mark - NSWeakProxy Method Overrides
#pragma mark Handling Unimplemented Methods

- (void)forwardInvocation:(NSInvocation *)invocation
{
    // Fallback for when target is nil. Don't do anything, just return 0/NULL/nil.
    // The method signature we've received to get here is just a dummy to keep `doesNotRecognizeSelector:` from firing.
    // We can't really handle struct return types here because we don't know the length.
    void *nullPointer = NULL;
    [invocation setReturnValue:&nullPointer];
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)selector
{
    // We only get here if `forwardingTargetForSelector:` returns nil.
    // In that case, our weak target has been reclaimed. Return a dummy method signature to keep `doesNotRecognizeSelector:` from firing.
    // We'll emulate the Obj-c messaging nil behavior by setting the return value to nil in `forwardInvocation:`, but we'll assume that the return value is `sizeof(void *)`.
    // Other libraries handle this situation by making use of a global method signature cache, but that seems heavier than necessary and has issues as well.
    // See https://www.mikeash.com/pyblog/friday-qa-2010-02-26-futures.html and https://github.com/steipete/PSTDelegateProxy/issues/1 for examples of using a method signature cache.
    return [NSObject instanceMethodSignatureForSelector:@selector(init)];
}
```

