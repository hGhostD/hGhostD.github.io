---
layout: RxSwift 实现 UITableview
title: RxSwift 实现 UITableview
date: 2018-01-26 13:48:02
tags: 
- Swift
categories: 
- Swift
---
UITableView 在开发中是最常使用的控件，由于 UITableview 相对来说功能比较多，所以对应的方法也很多，实现操作起来比较繁琐。通过 RxSwift 可以简化实现 UITableview 的方法，让开发者更注重业务逻辑。
<!----more----> 
#### 简单的实现方式
创建一个没有额外 Section 的 Tableview 可以很容易的实现。思路就是通过 Observable 将数据逐个绑定到相应的 Cell 上。

```
var dataArray = Variable<[model]>([])

struct Model {
    let title: String
}
```
接下来就是创建一个将 Model 绑定到 Cell 上的闭包

```
let setCell = {(i: Int, e: Model, c: TableViewCell) in
	c.setupWithModel(e)
}
```
准备工作都已经做完了，接下来就是最关键的绑定部分，通过 Observeable 的 bind 方法来实现：
`dataArray.asObservable().bind(to: self.tableView.rx.items(cellIdentifier: "Cell", cellType: TableViewCell.self))(setCell).disposed(by: bag)`
注意其中的 cellType 一定要和刚才创建的闭包中的对应，否则就会报错。

#### 带 Section 的实现方法
刚才的实现方式仅仅创建了一个 Section，很多时候是无法满足需求的。想创建带 Section 的 UITableview 就需要引入 `import RxDataSources` 来实现了。

首先，创建一个 dataSource 用来处理 UITableview 中的代理方法

```
let dataSource = RxTableViewSectionedReloadDataSource<SectionModel<String, Model>>(configureCell:{ (source, tableview, index, model) -> UITableViewCell in
	let cell = tableview.dequeueReusableCell(withIdentifier: "Cell", for: index) as! TableViewCell
	cell.setupWithModel(model)
	return cell
})
```
由于需要使用 Section，所以我们的数据源也要进行一些修改：
`var dataArray = Variable<[SectionModel<String, Model>]>([])`。
接下来的绑定和上边的方式差不多，我们可以直接将 dataSource 绑定到 dataArray 上。

`dataArray.asObservable().bind(to: self.tableView.rx.items(dataSource: dataSource)).disposed(by: bag)`

#### Proxy
RxSwift 已经提供了很多代理方法以供我们使用，但是有的时候我们还是需要创建自己的代理方法。
创建一个新文件来处理 Proxy ：

```
// 泊学网的方法还是 Swift 2.0 的版本 新版的 RxSwift 已经做了一些修改
class MyDelegateProxy: DelegateProxy<UITableView, UITableViewDelegate> ,UITableViewDelegate ,DelegateProxyType {
    
    public weak private(set) var tableView: UITableView?
    
    public init(tableView: ParentObject) {
        self.tableView = tableView
        super.init(parentObject: tableView, delegateProxy: MyDelegateProxy.self)
    }
	 // 如果没有实现这个方法 下面两个方法也不会实现 就会报错！参考了 Rx 中的实现方式实现的
    static func registerKnownImplementations() {
        self.register { MyDelegateProxy(tableView: $0) }
    }
    
    static func setCurrentDelegate(_ delegate: UITableViewDelegate?, to object: UITableView) {
        object.delegate = delegate
    }
    
    static func currentDelegate(for object: UITableView) -> UITableViewDelegate? {
        return object.delegate
    }
}
```
接下来需要对 UITableview 进行拓展

```
// 对 Selector 进行处理可以优雅的调用
private extension Selector {
    static let didSelectRowAtIndexPath = #selector(UITableViewDelegate.tableView(_:didSelectRowAt:))
}

extension UITableView {
    var rxDelegate: MyDelegateProxy {
        return MyDelegateProxy.proxy(for: self)
    }
    // RxSwift 中对 Selector 绑定的方法已经修改 这里需要使用 methodInvoked 方法实现。
    var rxDidSelectRowAtIndexPath: Observable<(UITableView, IndexPath)> {
        return rxDelegate.methodInvoked(.didSelectRowAtIndexPath).map { a in
            return (a[0] as! UITableView, a[1] as! IndexPath)
        }
    }
}
```
等以上准备步骤完成以后，接下来的调用就很容易了：

```
tableView.rxDidSelectRowAtIndexPath.subscribe(onNext: { a, b in
	print(b)			// [0,1]
}).disposed(by: bag)   
```
有的时候我们同样需要原生的代理方法，RxSwift 也对此进行了支持。不过不能简单的使用 `tableView.delegate = self` 来实现了。需要调用 `tableView.rx.setDelegate(self).disposed(by: bag)` 来设置代理人。**注意:委托代理一定要在绑定数据前执行，否则会造成崩溃！**
直接对 ViewController 进行拓展就能够实现原生代理方法了：

```
extension ViewController {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        // 视频中说 会先走 Proxy 中的方法，但是实践的时候我发现情况相反。先走原生代理方法，然后再执行 Proxy 中的订阅方法。
        print(indexPath)
    }
}
```
#### 关于下拉刷新
下拉刷新这个功能很常见，这里就简单记录一下我的实现方法吧。这里使用三方框架 `MJRefresh`。
首先创建下拉和上拉对象

```
// 顶部刷新
let header = MJRefreshNormalHeader()
// 底部刷新
let footer = MJRefreshAutoNormalFooter()    
```
然后绑定好处理事件

```
header.setRefreshingTarget(self, refreshingAction: #selector(upRefresh))
footer.setRefreshingTarget(self, refreshingAction: #selector(downRefresh))
self.tableView.mj_header = header
self.tableView.mj_footer = footer
```
最后实现刷新事件即可

```
@objc func upRefresh() {
    dataArray.value.removeAll()
    page = 1
    Network.default.searchDouBan(start: String(page), count: "10").subscribe(onNext:{
        self.tableView.mj_header.endRefreshing()
        // 封装成 SectionModel 数据绑定
		 let sec = SectionModel(model: "header", items: $0)
        self.dataArray.value.append(sec)    
    }).disposed(by: bag)
}

@objc func downRefresh() {
    page += 1
    Network.default.searchDouBan(start: String(page), count: "10").subscribe(onNext:{
        self.tableView.mj_footer.endRefreshing()
        let sec = SectionModel(model: "header", items: $0)
        self.dataArray.value.append(sec)  
    }).disposed(by: bag)
}
```