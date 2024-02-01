---
layout: Swift
title: 使用 UIPresentationController 实现底部弹出页面 
date: 2018-03-08 14:07:13
tags:
- Swift
categories:
- Swift
---
底部弹出框在项目中很常见，之前对此的理解很肤浅，不太明白如何更好的实现自定义的页面。刚好在 [GitHub](https://github.com/IkeBanPC/PresentBottom) 看到有一中很简单的实现方式，是利用 UIPresentationController 和继承的方式实现。学习一下，自己再通过协议的方式实现下，加深自己的理解。
<!-- more -->
#### 实现效果
![UIPresentationController 底部弹出](https://raw.githubusercontent.com/IkeBanPC/PresentBottom/master/Pics/Select.gif)
#### 实现 UIPresentationController
关于 `UIPresentationController` 的描述，官网的说法是 
> An object that manages the transition animations and the presentation of view controllers onscreen.

简单点说就是管理两个 Controller 之间的转场动画。
所以可以通过重写这个类来自定义想要的转场动画。
首先创建一个类来继承 `UIPresentationController`，并重写一些必要的方法

```
class HUPresentController: UIPresentationController {
	//获取转场 Controller 高度
    var controllerHeight: CGFloat = 0
    //决定了弹出框的frame
    override var frameOfPresentedViewInContainerView: CGRect {
        return CGRect(x: 0, y: UIScreen.main.bounds.height - controllerHeight , width: UIScreen.main.bounds.width, height: controllerHeight)
    }
    //重写此方法可以在弹框即将显示时执行所需要的操作
    override func presentationTransitionWillBegin() {
        blackView.alpha = 0
        containerView?.addSubview(blackView)
        UIView.animate(withDuration: 0.3) { self.blackView.alpha = 1 }
    }
    //重写此方法可以在弹框显示完毕时执行所需要的操作
    override func presentationTransitionDidEnd(_ completed: Bool) {
    }
    //重写此方法可以在弹框即将消失时执行所需要的操作
    override func dismissalTransitionWillBegin() {
        UIView.animate(withDuration: 0.5) { self.blackView.alpha = 0 }
    }
    //重写此方法可以在弹框消失之后执行所需要的操作
    override func dismissalTransitionDidEnd(_ completed: Bool) {
        if completed { blackView.removeFromSuperview() }
    }
    //遮挡阴影
    lazy var blackView: UIView = {
        let view = UIView()
        if let frame = self.containerView?.bounds {
            view.frame = frame
        }
        view.backgroundColor = UIColor.black.withAlphaComponent(0.5)
        return view
    }()

    override init(presentedViewController: UIViewController, presenting presentingViewController: UIViewController?) {
        if let vc = presentedViewController as? PresentProtocol {
            controllerHeight = vc.controllerHeight
        }
        super.init(presentedViewController:presentedViewController,presenting: presentingViewController)
    }
}
```
#### 协议 && UIViewController 扩展
原来的项目是通过继承的方式来实现的，但是在 Swift 中并不提倡使用继承，所以我们改用协议的方式来实现。
```
protocol PresentProtocol {
    var controllerHeight: CGFloat { get }
}
extension PresentProtocol where Self: UIViewController {}
//对 UIViewController 进行拓展，并让其遵守转场协议
extension UIViewController: UIViewControllerTransitioningDelegate  {
    func presentBottom(_ vc: UIViewController) {
        vc.modalPresentationStyle = .custom
        vc.transitioningDelegate = self
        self.present(vc, animated: true, completion: nil)
    }
    public func presentationController(forPresented presented: UIViewController, presenting: UIViewController?, source: UIViewController) -> UIPresentationController? {
        let present = HUPresentController(presentedViewController: presented, presenting: presenting)
        return present
    }
}
```
#### 实现自定义 Controller 
准备工作到了最后一步就是实现自定义的 Controller 了，确保 Controller 遵循刚才定义的协议 `PresentProtocol`，并实现协议属性。

```
class HUTestVC: UIViewController, PresentProtocol {
    var controllerHeight: CGFloat { return 600 }
    /// sureButton to hide bottom vc
    lazy var sureButton:UIButton = {
        let button = UIButton(frame: CGRect(x: kScreenWidth-60, y: 0, width: 40, height: 40))
//        button.setImage(, for: .normal)
        button.backgroundColor = .white
        button.addTarget(self, action: #selector(sureButtonClicked), for: .touchUpInside)
        button.layer.cornerRadius = 20
        button.clipsToBounds = true
        return button
    }()
    
    /// conntainer view of bottom vc
    lazy var containerView: UIView = {
        let view = UIView(frame: CGRect(x: 0, y: 75, width: kScreenWidth, height: kScreenHeight-75))
        view.backgroundColor = UIColor.white
        return view
    }()
    /// titleLabel
    lazy var titleLabel: UILabel = {
        let label = UILabel(frame:CGRect(x: (kScreenWidth-150)/2, y: 20, width: 150, height: 30))
        label.textAlignment = .center
        label.text = "Select"
        label.font = UIFont.systemFont(ofSize: 20)
        return label
    }()
    override public func viewDidLoad() {
        super.viewDidLoad()
        config()
    }
    private func config() {
        view.backgroundColor = UIColor.clear
        let roundView = RoundView(frame: CGRect(x: 0, y: 0, width: kScreenWidth, height: 150))
        view.addSubview(roundView)
        roundView.addSubview(titleLabel)
        view.addSubview(containerView)
        view.addSubview(sureButton)
        let segment1 = UISegmentedControl(items: ["Girl","Boy","Unsure"])
        segment1.frame = CGRect(x: 20, y: 20, width: kScreenWidth-40, height: 35)
        segment1.selectedSegmentIndex = 0
        segment1.tintColor = UIColor(red: 190/255, green: 31/255, blue: 109/255, alpha: 1)
        containerView.addSubview(segment1)
        let segment2 = UISegmentedControl(items: ["🍎","🍋","🍊"])
        segment2.frame = CGRect(x: 20, y: 75, width: kScreenWidth-40, height: 35)
        segment2.selectedSegmentIndex = 1
        segment2.tintColor = UIColor(red: 190/255, green: 31/255, blue: 109/255, alpha: 1)
        containerView.addSubview(segment2)
        let segment3 = UISegmentedControl(items: ["Home","Company","Parking Lot"])
        segment3.frame = CGRect(x: 20, y: 130, width: kScreenWidth-40, height: 35)
        segment3.selectedSegmentIndex = 2
        segment3.tintColor = UIColor(red: 190/255, green: 31/255, blue: 109/255, alpha: 1)
        containerView.addSubview(segment3)
    }
    @objc func sureButtonClicked() {
        self.dismiss(animated: true, completion: nil)
    }
}
/// create a bezier path view
public class RoundView: UIView {
    public override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = UIColor.clear
    }
    required public init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }
    public override func draw(_ rect: CGRect) {
        let color = UIColor.white
        color.set()
        let path = UIBezierPath(ovalIn: rect)
        path.fill()
    }
}
```

#### 最后调用
调用和系统调用类似，在第一个 Controller 上调用 `self.presentBottom(HUTestVC())`。
<br>
> 参考资料
> [用UIPresentationController来写一个简洁漂亮的底部弹出控件](https://github.com/IkeBanPC/PresentBottom)