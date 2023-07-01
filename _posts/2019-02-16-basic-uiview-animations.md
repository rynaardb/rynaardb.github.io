---
layout: post
title: "Basic UIView Animations"
date: 2019-02-16
categories: ios
tags: ios swift
image:
    path: /assets/images/uiview-animations.png
---

Subtle and concise animations can help draw the user’s attention to a specific area of your application. Often it may be unclear to the user when to perform a certain action or as to what is going on after a specific task / action was completed. This is where well thought out animations can be very useful.

## UIView.animate

There are several different APIs you can use to create animations but the most common and also very easy to use API is [UIView.animate](https://developer.apple.com/documentation/uikit/uiview/1622418-animate).

Animating properties on a view can be done by calling UIView.animate where you need to specify the TimeInterval (duration) the animation will take to complete. Below simply animates the alpha channel of the view from 1 (visible) to 0 (completely transparent):

```swift
func animateToothless() {
    UIView.animate(withDuration: 1) {
        self.leftTooth.alpha = 0
        self.rightTooth.alpha = 0
    }
}
```

The example above demonstrates a simple once-off animation but more often than not you would need to set some properties or do something else once the animation completes. For this there is another overload providing a completion callback once the animation completes:

```swift
func animateWink() {
    let originalHeight = rightEye.frame.size.height
    let originalYPos = rightEye.center.y

    UIView.animate(withDuration: 0.3, animations: {
        self.rightEye.frame.size.height = 0
        self.rightEye.center.y = 400
    }) { (true) in
        self.rightEye.frame.size.height = originalHeight
        self.rightEye.center.y = originalYPos
    }
}
```

Above captures the original height and y position of the view before the animation starts and later on sets the view's properties back to the original values once the animation completes.

Additionally, one can also animate views using a timing curve corresponding to the motion of a physical spring. With this overload you can get rather creative by making very unique and realistic animations:

```swift
func animateToungh() {
    let originaHeight = toungh.frame.size.height

    UIView.animate(withDuration: 1,
                   delay: 0,
                   usingSpringWithDamping: 0.2,
                   initialSpringVelocity: 5,
                   options: .curveEaseOut,
                   animations: {
        self.toungh.frame.size.height = 55
    }) { _ in
        self.toungh.frame.size.height = originaHeight
    }
}
```

When used correctly animations can delight your users and add overall polish to your app.

A copy of the Xcode Playground can be found on my GitHub [repository](https://github.com/rynaardb/ios-animations).
