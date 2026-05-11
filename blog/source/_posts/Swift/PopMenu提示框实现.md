---
layout: swift
title: PopMenu提示框实现
date: 2018-05-11 15:15:49
tags: 
- Swift
categories: 
- Swift
---
在网上看到了一款实现效果不错的提示框，可以自定义提示框样式，设置提示框出现位置。决定看一下别人是如何实现的，了解下别人的实现思路。
<!-- more -->
#### 实现效果
效果如下图所示。
![](https://raw.githubusercontent.com/CaliCastle/PopMenu/master/.assets/Demo_Showcase.gif)
使用起来也很简单，[文档](https://calicastle.github.io/PopMenu/index.html)也有很仔细的说明，这里就无需在多做说明。

#### 代码解读
具体化还是看一下 PopMenu 是如何实现的。借鉴一下别人的实现思路。
##### 初始化类型
PopMenu 使用了两种方式进行实例化对象。第一种是用 Manager 创建单例对象，第二种是创建 Controller 进行使用。
这里我们先研究下第一种实现方式。

```
final public class PopMenuManager: NSObject {
    /// 默认的管理对象
    public static let `default` = PopMenuManager()
    /// 放置 Action 数组
    public var actions: [PopMenuAction] = []
    ···	
}
```
这种方式可以通过直接调用静态属性 `default`。创建对象，然后添加 `actions` 中的内容。可以清楚的看到 `action` 中需要存放遵循 `PopMenuAction` 协议的对象。这样就可以通过实现协议来自定义 action 的 UI。
##### 自定义 Action
只要遵循实现 `PopMenAction` 协议，就可以实现自定义的 Action。让我们看一下默认的的 `PopMenuDefaultAction` 是如何实现的。首先是 `PopMenuAction` 协议，这里就只列出简单的属性。

```
/// Image of the action.
@objc public protocol PopMenuAction: NSObjectProtocol {
    var image: UIImage? { get }
    var view: UIView { get }
    ······
    func renderActionView()
}
```

当协议完成了，就可以创建一个 class 来遵循这个协议，实现默认的 Action。

```
public class PopMenuDefaultAction: NSObject, PopMenuAction {
    public let image: UIImage?
    public let view: UIView
    ······
    /// 初始化方法
    public init(image: UIImage? = nil) {
        self.image = image
        view = UIView()
    }
    /// 加载渲染的 view
    public func renderActionView() {
        /// 自定义 UI
        configureViews()
    }
}
```

默认的 Action 实现好了，使用就比较简单了，直接将实例对象赋到数组中就行。

```
manager.actions = [
    PopMenuDefaultAction(title: "标题一"),
    PopMenuDefaultAction(title: "有图片的标题", image: #imageLiteral(resourceName: "clear-day"), color: UIColor.orange, didSelect: complete)
]
```

##### 显示 & 消失 提示框
准备工作都完成了，就可以在需要的地方调用显示方法就行。

`manager.present(sourceView: button, on: self, animated: true)`
一句话即可实现，让我们看看到底是怎么完成这个显示的，其中包括了，转场，显示动画和消失的动画。

```
public func present(sourceView: AnyObject? = nil, on viewController: UIViewController? = nil, animated: Bool = true, completion: (() -> Void)? = nil) {

        prepareViewController(sourceView: sourceView)
        guard let popMenu = popMenu else { print("Pop Menu has not been initialized yet."); return }
        
        if let presentOn = viewController {
            presentOn.present(popMenu, animated: animated, completion: completion)
        } else {
            if let topViewController = PopMenuManager.getTopViewControllerInWindow() {
                topViewController.present(popMenu, animated: animated, completion: completion)
            }
        }
    }
```

看起来很简单的实现方式，首先准备了一个 VC `prepareViewController(sourceView: sourceView)`。然后使自身 `popMenu` 持有该属性。然后获取 VC 显示 `popMenu`。

关键就在于如何实现这个准备用的 VC `PopMenuViewController`了。

观察代码可以发现，`PopMenuViewController` 这个自定义的 VC 实现了提示框的 UI 设置与手势交互。

```
final public class PopMenuViewController: UIViewController {
    public let actionsView = UIStackView()
    
    fileprivate func configureActionsView() {
    	//// 给 actionsView 添加 具体的 View 并做好布局
    }
    ······
}
```

整体实现的思路还是很清晰的，具体的细节有很多，就不一一列举了。
关于显示 VC，让 `PopMenuViewController ` 遵循转场动画协议 `UIViewControllerTransitioningDelegate` 并实现它。


```
extension PopMenuViewController: UIViewControllerTransitioningDelegate {
    
    public func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return PopMenuPresentAnimationController(sourceFrame: absoluteSourceFrame)
    }
    
    public func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return PopMenuDismissAnimationController(sourceFrame: absoluteSourceFrame)
    }
}
···
final public class PopMenuPresentAnimationController: NSObject, UIViewControllerAnimatedTransitioning {
	/// 实现代理方法 自定义细节动画
}
```

关于消失，实现思路就很简单了，在 VC 的背景 View 中添加手势，并实现系统点击事件 `dismiss()`，消失整个 VC 即可。

#### 总结
这个 PopMenu 实现思路并不是很复杂，但是其中的细节做得很到位，效果也很好，文档也写的很棒。让我对自定义提示框的理解学习到了很多。
<br>
>参考资料
>[PopMenu Git](https://github.com/CaliCastle/PopMenu)
>[PopMenu使用文档](https://calicastle.github.io/PopMenu/index.html)
