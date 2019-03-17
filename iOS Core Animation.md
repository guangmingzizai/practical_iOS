# iOS Core Animation

Modern, fast and fluid user interface.

`LayerKit`，developed by the iPhone software team to provide a high-performance, hardware-accelerated animation and compositing library to replace the slower, Quartz-based software drawing used by AppKit. This framework actually debuted first on Mac OS 10.5 under the name Core Animation, shortly before the iPhone was announced.

Core Animation is not just a set of functions for performing animations; it lies at the very heart
of iOS, powering everything you see on screen.

Core Animation is a *compositing engine*; its job is to compose different pieces of visual content on the screen, and to do so as fast as possible.

Rendering, layout, and animation are all managed by a Core Animation class called CALayer.

The only major feature of UIView that isn’t handled by CALayer is user interaction.

*Four* hierarchies, each performing a different role. In addition to the view hierarchy and layer tree, there are the *presentation tree* and *render tree*.

The lightweight UIView class in iOS barely has any negative impact on performance when working with layers. (In Mac OS 10.8, the performance of NSView is greatly improved, as well.)

most visual properties of UIView—such as contentMode—are really just manipulating equivalent properties of the underlying layer.

Typically, many sprites will be packed into a single large image that can be loaded in one go. This carries various benefits over using multiple individual image files in terms of memory usage, load time, and rendering performance.

The perspective effect of a CATransform3D is controlled by a single value in the matrix: element m34. The m34 value is used in the transform calculation to scale the X and Y values in proportion to how far away they are from the camera.

By default, m34 has a value of zero. We can apply perspective to our scene by setting the
m34 property of our transform to -1.0 / *d*, where *d* is the distance between the imaginary
camera and the screen, measured in points. How do we calculate what this distance should
be? We don’t have to; we can just *make something up*.

Because the camera doesn’t really exist, we are free to decide where it is positioned based
on what looks good in our scene. A value between 500 and 1000 usually works fairly well,
but you may find that smaller or larger values look better for a given layer arrangement.
Decreasing the distance value increases the perspective effect, so a very small value will
look extremely distorted, and a very large value will just look like there is no perspective at
all (isometric).

When you change the position of a layer, you also change its vanishing point. This is
important to remember when you are working in 3D. If you intend to adjust the m34
property of a layer to make it appear three-dimensional, you should position it in the center
of the screen and then move it to its final location using a translation (instead of changing
its position) so that it shares a common vanishing point with any other 3D layers on the
screen.

Although Core Animation layers exist in 3D space, they don’t all exist in *the same* 3D space. The 3D scene within each layer is flattened. When you look at a layer from face on, you see the *illusion* of a 3D scene created by its sublayers, but as you tilt the layer away, you realize that 3D scene is just painted on the layer surface.

A CATransformLayer is unlike a regular CALayer in that it cannot display any content of its own; it exists only to host a transform that can be applied to its sublayers. CATransformLayer does not flatten its sublayers, so it can be used to construct a hierarchical 3D structure.

## 术语

*Responder chain*: the mechanism that iOS uses to propagate touch events through the view hierarchy.

*backing layer*: view's layer. The view is responsible for creating and managing this layer and for
ensuring that when subviews are added or removed from the view hierarchy that their corresponding backing layers are connected up in parallel within the layer tree.

*presentation tree*:

*render tree*: 

*Unit coordinates* are specified in the range 0 to 1, and are *relative* values (as opposed to *absolute* values like points and pixels). 

*Sprite sheets*: image maps containing multiple sub-images.

Affine transform: lines in the layer that were parallel before the transform will remain parallel after the transform.

isometric projection: objects that are farther away appear at the same scale as objects
that are close to us. 

## 知识点

CGImage不包含scale信息。

A single large image compresses better and loads quicker than multiple small ones).

`frame`是一个虚拟属性，由`bounds`、`center/position`、`transform`计算而来。When a layer is rotated or scaled, its frame reflects the total axially aligned rectangular area occupied by the transformed layer within its parent, which means that the frame width and height may no longer match the bounds.

修改`anchorPoint`时，`bounds`和`position`保持不变，`frame`会改变。

Unlike UIView, which is strictly two-dimensional, CALayer exists in three-dimensional space. Views are infinitely thin.

Layers are fundamentally flat objects. You can think of them as being a bit like stiff sheets of paper that are individually two-dimensional but that can be glued together to form hollow, origami-like 3D structures.

`cornerRadius`默认只影响`backgroundColor`，而不影响backing image和子view。

The border is drawn *inside* the layer bounds, and in front of any other layer contents, including sublayers.

Layer border follows the bounds of the layer, including the corner curvature.

Unlike the layer border, the layer’s shadow derives from the *exact* shape of its contents, not
just the bounds and cornerRadius.

When you apply transforms sequentially in this way, the previous transforms affect the subsequent ones. 例如：

```objective-c
//create a new transform
CGAffineTransform transform = CGAffineTransformIdentity; //scale by 50%
transform = CGAffineTransformScale(transform, 0.5, 0.5); //rotate by 30 degrees
transform = CGAffineTransformRotate(transform, M_PI / 180.0 * 30.0); //translate by 200 points
transform = CGAffineTransformTranslate(transform, 200, 0);
//apply transform to layer
self.layerView.layer.affineTransform = transform;
```

The 200-point translation to the right has been rotated by 30 degrees and scaled by 50%, so it has actually become a translation diagonally downward by 100 points.

The order in which you apply transforms affects the result; a translation followed by a rotation is not the same as a rotation followed by a translation.

## Questions

**研究方法是什么？如何透彻研究Core Animation这样的技术？**

**Cocoa Touch、UIKIt、Core Animation、Core Graphics等的作用？彼此关系？**

**为什么把view和layer分开？**

The reason is to separate responsibilities, and so avoid duplicating code. Events and user interaction work quite differently on iOS than they do on Mac OS; a user interface based on multiple concurrent finger touches (*multitouch*) is a fundamentally different paradigm to a mouse and keyboard, which is why iOS has UIKit and UIView and Mac OS has AppKit and NSView. They are functionally similar, but differ significantly in the implementation.

**有哪些CALayer的功能没有通过View暴露出来？**

- Drop shadows, rounded corners, and colored borders
- 3D transforms and positioning
- Nonrectangular bounds
- Alpha masking of content 
- Multistep, nonlinear animations

**哪些情况下需要直接使用CALayer，而不是使用UIView/NSView？**

- You might be writing cross-platform code that will also need to work on a Mac. 
- You might beworking with multiple CALayer subclasses(see Chapter6, “Specialized Layers”) and have no desire to create new UIView subclasses to host them all. 
- You might be doing such performance-critical work that even the negligible overhead of maintaining the extra UIView object makes a measurable difference (although in that case, you’ll probably want to use something like OpenGL for your drawing anyway). 

**为什么CALayer的contents属性是id类型？**

历史遗留问题。在Mac OS上，contents可以被设置为CGImage或NSImage（iOS上，只能设置为CGImage，否则内容为空）。

**iOS是如何根据屏幕分辨率显示不同尺寸图片的？**

UIImage会根据UIScreen.main.scale加载不同的图片文件，并设置自己的scale属性。Layer显示时，contentsScale设置为图片的scale，contents设置为image.cgImage即可。

概括来说就是，根据UIScreen.main.scale加载不同的图片，设置layer.contentsScale为UIScreen.main.scale。



```objective-c
// CALayer
- (void)display {
    if ([delegate respondsToSelector(@selector(displayLayer:))]) {
        [delegate displayLayer:self];
    } else {
        if ([delegate respondsToSelector(@selector(layerWillDraw:))]) {
            [delegate layerWillDraw:self];
        }
        UIGraphicsBeginImageContextWithOptions(self.bounds.size, NO, UIScreen.mainScreen.scale);
        CGContextRef ctx = UIGraphicsGetCurrentContext();
        [self drawInContext:ctx];
        UIImage *backingImage = UIGraphicsGetImageFromCurrentImageContext();
        self.contents = (__bridge_retained id)backingImage.cgImage;
        UIGraphicsEndImageContext();
    }
}

- (void)drawInContext:(CGContextRef)cxt {
    if ([delegate respondsToSelector(drawLayer:inContext:)]) {
        [delegate drawLayer:self inContext:ctx];
    }
}

// UIView
- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)cxt {
    [self drawRect:self.bounds];
}
```

