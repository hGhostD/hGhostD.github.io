---
layout: swift
title: 关于WKWebView和JS交互开发
date: 2017-11-03 09:18:58
tags:
- Swift
categories:
- Swift
---
在工作中使用到了 WKWebView 和 JS 交互开发的问题，在此留作笔记。
<!--- more --->
#### 引言
关于 WKwebview 的优点就不多说了，苹果公司也推荐使用 WKWebView 来代替 UIWebView 作为网页浏览器开发。正好遇到的项目有这一部分需求，就直接使用了，也顺便学习一下，将遇到的问题记录下来。
#### 设置WKWebView
最简单的第一步还是需要引入头文件 `import WebKit`

为了能与 JS 进行交互，需要对 WKWebView 进行设置


```
//创建配置
let config = WKWebViewConfiguration()
// 创建UserContentController（提供JavaScript向webView发送消息的方法）
let userContent = WKUserContentController()
// 添加消息处理，注意：self指代的对象需要遵守WKScriptMessageHandler协议，结束时需要移除
userContent.add(self, name: "webViewApp")
// 将UserConttentController设置到配置文件
config.userContentController = userContent

let frame = CGRect(x: 0, y: 0, width: SCREEN_WIDTH, height: SCREEN_HEIGHT - 64)
webView = WKWebView(frame: frame, configuration: config)
webView.uiDelegate = self           //WKUIDelegate
webView.navigationDelegate = self   //WKNavigationDelegate
```
注意，这里的 `userContent.add(self, name: "webViewApp")` 其中的 `webViewApp` 是与前端JS规定好的关键字，可以自行定义。

至此，关于 WKWebView 的基本设置已经完成，我们需要实现相关的代理方法。
#### 实现代理方法
一些基本的代理就不做特殊说明了。
##### WKNavigationDelegate
之前一直使用旧的 webView 加载完成的代理方法，还奇怪怎么一直不生效。后来才发现 WKWebView 需要使用 `webView(_ webView: WKWebView, didFinish navigation: WKNavigation!)` 方法才行。加载失败的代理方法同理。
##### WKScriptMessageHandler
当 JS 调用 WKWebView 时需要 JS 实现 ```
window.webkit.messageHandlers.webViewApp.postMessage(value);
```
其中 `webViewApp` 是我们在开始注册的关键字。

为了接受 JS 调用必须实现这个协议和其中的代理方法 `userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage)`
并且可以从 `message` 中获取由 JS 传递过来的 JSON 信息。
`let dict = message.body as? Dictionary<String,String>`
例：从dict中取值 `if let pageId = dict?["pageId"] { K_CONFIGJSON.pageId = pageId }`
后来发现用 SwiftyJson 框架处理 JSON 信息更容易些，我就换成了下面的处理方法:
```
let dict = JSON(message.body)
K_CONFIGJSON.pageId = dict["pageId"].stringValue
```
在此方法中就可根据需求实现自定义方法。
#### WKWebview 调用 JS 方法
```
DispatchQueue.main.async {
	let email = K_USERMODEL.email
	let password = K_USERMODEL.password
 	self.webView.evaluateJavaScript("nativeCallBack('\(email),\(password)')") { (item, error) in
		print(item ?? "没有返回",error ?? "没有错误")
	}
}
```
注意：这里调用 JS 方法一定要在主线程调用，不然有时会造成崩溃！！！
最初我发现调用 JS 的方法如果添加了参数会导致方法失效，参数不能传递过去。通过查阅资料发现了可以通过前端更改代码来解决问题，参考资料在最后。

#### 最后还有一点
本来还想在 XCode 的调试窗口里显示出 JS 的 `console.log()` 信息，因为这个功能在 Android 上是自带的。但是发现还需要自己集成，太麻烦了。。。所以我就放弃了。不过发现了一篇关于这个的文章，就先放这里吧。
<br>

> 参考资料
> [WK 与 JS 的那些事](http://www.jianshu.com/p/c9ceb6a824e2)
> [实现 js 向 Swift 的传值](https://lvwenhan.com/ios/462.html)
> [Swift-On-iOS](https://github.com/johnlui/Swift-On-iOS/tree/master/BuildYourOwnHybridDevelopmentFramework/BuildYourOwnHybridDevelopmentFramework)