---
layout: swift
title: Swift协议拓展
date: 2017-11-20 16:29:11
tags: 
- Swift
categories: 
- Swift
---
很多时候都会对 UI 进行自定义设置，之前使用 Objective-C 的时候都是用分类来自定义方法，而 Swift 提供了拓展可以更方便的来处理。看过一篇关于 UI 链式编程的博客，发现实现思路很棒，这里来学习一下。
<!-- more -->
#### 简单实现方式
要对 UI 进行拓展可以直接使用 extension 添加自定的方法。但是这么做的话感觉不够帅气！

```
extension UIView: ViewChainable {
    func adhere(_ toSupView: UIView) -> Self {
        toSupView.addSubview(self)
        return self
    }
    // 使用 @discardableResult 关键字可以让 XCode 忽略返回值警告
    @discardableResult
    func layout(snapKitMaker: (ConstraintMaker) -> Void) -> Self {
        self.snp.makeConstraints {
            snapKitMaker($0)
        }
        return self
    }
}
```
这样就给算给 UIView 添加了 `adhere()` 和 `layout()` 方法。虽然只是给父视图添加自身和 SnapKit 布局，但是简单的思路就是在 extension 中添加自定义方法。
假设我们现在需求如下：创建一个 UILabel，背景色为透明，字体为20，文字颜色为黑灰，默认文字为 Label。
如果我们用刚才的思路就是在 extension 中添加一个 `config()` 方法把我们刚才的设定都放进去，然后创建一个 UILabel 调用拓展方法。这么做当然可以，但是不够灵活也不够帅气！！！在 Swift 中可以使用闭包和协议，这些问题就能够很好的解决。
首先定义一个空的协议。然后对这个协议进行拓展，并且限定 UIView 遵循这个协议。最后就能够在拓展里添加方法，方法的参数是一个闭包，传递自身。

```
protocol ViewChainable {}
extension ViewChainable where Self: UIView {
    @discardableResult
    func config(_ config: (Self) -> Void) -> Self {
        config(self)
        return self
    }
}
```
这些协议和拓展设置完成就算是能够对 UI 进行链式编程了。简单的实现如下:

```
let label = UILabel()
    .adhere(toSuperView: view)
    .layout { (make) in
        make.top.equalToSuperview().offset(80)
        make.centerX.equalToSuperview()
    }
    .config { (label) in
        label.backgroundColor = UIColor.clear
        label.font = UIFont.systemFont(ofSize: 20)
        label.textColor = UIColor.darkGray
        label.text = "Label"
    }
```
关于我们的需求，看起来好像没有更好的处理，也需要在每个 lable 中进行单独的设置。但是不要忘记了，我们的 `config()` 方法中传递的是一个**闭包**，也就是说可以抽象出设置方法作为参数传递给多个 label。

```
let labelConfiger = { (label: UILabel) in
    label.backgroundColor = UIColor.clear
    label.font = UIFont.systemFont(ofSize: 20)
    label.textColor = UIColor.darkGray
    label.text = "Label"
}
label.config(labelConfiger)
```
这个时候如果还有一个 label2。我们就可以给 label2 直接传递定义好的闭包 `labelConfiger`。如果 label2 的 text 需要传递不同的内容，实现起来也很容易，只需要再次传递一个 `config()` 就可以了。

```
label.config(labelConfiger)
    .config { (label) in
        label.text = "Label1"
    }

label2.config(labelConfiger)
    .config { (label) in
        label.text = "Label2"
    }
```
其实做到这里，也会发现**链式函数的关键就在于<font color=#d13f28>返回值类型与自身相同</font>**。

#### 很帅气的实现方式
在使用 RxSwift 的时候就觉得自定义 .rx 的方式很帅，但是一直不知道如何实现的。知道看过了牛人的 blog 才发现实现起来蛮简单的，但是不那么好理解，接下来就按照他的思路一步步整理一下。
首先肯定是定义一个协议，这个协议的作用就是对需要封装的内容进行拓展，定义一个自定义参数。这里使用 `hu` 作为拓展属性的名称。

```
public protocol HUNamespaceProtocol {
    /// - associatedtype 关键字 用来在协议中表达参数化类型
    /// - 定义 HUNameType 为协议中的参数类型
    associatedtype nameType
    var hu: nameType { get }
    static var hu: nameType.Type { get }
}
```
Tips: 协议中不能直接使用 `Element` 作为泛型关键字，需要时用 `associatedtype` 声明一个形参。
接下来就需要对这个协议进行拓展。

```
public extension HUNamespaceProtocol {
    var hu: HUNamespaceWrapper<Self> {
        return HUNamespaceWrapper(value: self)
    }
    static var hu: HUNamespaceWrapper<Self>.Type {
        return HUNamespaceWrapper.self
    }
}
public protocol TypeWrapperProtocol {
    associatedtype nameType
    var wrappedValue: nameType { get }
    init(value: nameType)
}
/// 定义的泛型结构体 遵循 TypeWrapperProtocol 协议
public struct HUNamespaceWrapper<T>: TypeWrapperProtocol {
    public let wrappedValue: T
    public init(value: T) {
        self.wrappedValue = value
    }
}
```
这里对协议的属性 `hu` 进行了拓展，定义了一个泛型结构体来用来接收初始化的对象作为自身。这里有一点难懂。因为这是基于一个这个原理的。
> namespace 形式扩展的原理，就是对原类型进行一层封装。在 Swift 中，这个封装类型使用的是 Struct，然后，对这个 Struct 进行自定义的方法扩展。

我的理解就是定义了一个泛型结构体，而这个泛型结构体遵循一个协议。这个协议规定了: 1.可以获取自身类型属性。2有一个使用自身属性初始化自身的方法。这样泛型结构体就可以通过 `wrappendVlaue` 属性来获取自身的 Type。这样当我们实现第一个协议的方法时候，规定属性 `hu` 必须继承我们定义的结构体，且通过规定好的初始化方法返回结构体对象。啊。。。还是很难懂。

接下来就是实现自定义的拓展方法了。这一步比较容易理解，只要让需要实现的类型遵循我们定义好的命名协议，然后在实现命名协议的扩展方法就可以了。例如下面分别给 UIView 和 String 添加了自定义方法。

```
extension UIView: HUNamespaceProtocol { }
///如果对象是引用类型的(类) 如: UIView 等 就必须使用 :
extension HUNamespaceWrapper where nameType: UIView {
    public func adhere(_ toSuperView: UIView) -> T {
        toSuperView.addSubview(wrappedValue)
        return wrappedValue
    }
    @discardableResult
    public func layout(_ snapKitMaker: (ConstraintMaker) -> Void) -> T {
        wrappedValue.snp.makeConstraints {
            snapKitMaker($0)
        }
        return wrappedValue
    }
    @discardableResult
    public func config(_ config: (T) -> Void) -> T {
        config(wrappedValue)
        return wrappedValue
    }
}
extension String: HUNamespaceProtocol {}
/// 如果约束对象是值类型的 如: String, Date 等 就必须使用 ==
extension HUNamespaceWrapper where nameType == String {
    func test(_ string: String) -> String {
        return "hu:\(string)"
    }
}
```
这样就可以很帅气的实现了。

```
print("".hu.test("123"))
let button = UIButton(type: .custom)
button.hu.adhere(self.view)
    .hu.config(buttonConfig)
    .hu.layout {
        $0.center.equalTo(self.view)
        $0.width.equalTo(120)
        $0.height.equalTo(40)
}
```
<br>
> 参考资料
> [Swift 实践篇之链式 UI 代码](https://blog.nswebfrog.com/2017/10/20/swift-practice-ui-chaining-code/)
> [Swift 命名空间形式扩展的实现](https://blog.nswebfrog.com/2017/03/23/swift-namespace/)