---
layout: Moya 使用
title: Moya 使用
date: 2018-02-06 16:54:57
tags: 
- Swift
categories: 
- Swift
---
Swift 经常用 Alamofire 来做网络库，而 Moya 在 Alamofire 的基础上又封装了一层。Moya 是你的 app 中缺失的网络层。不用再去想在哪儿（或者如何）安放网络请求，Moya 替你管理。
<!-- more -->
##### 安装配置
由于我在项目使用了 CocoaPods，而且也使用了 RxSwift 这个框架，所以直接在在 Podfile 文件中添加 `pod 'Moya/RxSwift'` 即可。
##### 创建 APIManager
[Moya 官方用例文档](https://github.com/Moya/Moya/blob/7c599570c2a60079dfefcc27fc67cb303feb48e1/docs_CN/Examples/Basic.md)讲解的很清晰，使用前我们需要创建一个枚举，作为网络请求的 API 请求分类。
先创建一个 APIManager.swift 用来统一处理的网络请求。然后创建 APIManager 枚举，用来区分不同的请求接口。

```
enum APIManager {
    case getNetWorkRegion  ///> 判断国内国外  0---国外  1---中国
}
```
##### 实现 TargetType 协议
接下来对这个 APIManager 进行拓展，并且让它遵循 TargetType 协议。这个协议是 Moya 为了方便创建一个 MoayProvider，对网络请求进行一个统一的管理。

```
extension APIManager: TargetType {
    var baseURL: URL { return URL(string: BASEURL)! }
    var path: String { return "" }
    var method: Moya.Method { return .post }
    var sampleData: Data { return "".data(using: .utf8)! }
    var headers: [String : String]? { return ["Content-type": "application/json"] }
    
    var task: Task {
        var dict = ["": ""]
        switch self {
        case .getNetWorkRegion:
            dict =  ["action": "ping"]
        }
        return .requestParameters(parameters: dict, encoding: URLEncoding.queryString)
    }
}
```
###### baseURL
基本的网络请求，所有网络请求都会经过这个 URL 进行封装。
###### path
具体的网络请求路径，由于我项目中所有的请求都是通过同一个路径，所以这个返回值我并不需要添加任何信息。
###### method
网络请求方法，例如 post，get。
###### sampleData
单元测试时需要使用的模拟数据，可以自定设置。这个 sample data 个是 Data 实例对象, 它可以表示 JSON, images, text, 或者任何您希望从endpoint得到的.
###### headers
请求头部数据。
###### task
由于最新的 Moya 已经修改，参数直接在这个属性中进行处理。注意：因为我之前的 encoding 中的参数填写的是 `URLEncoding.default` 和 `JSONEncoding.default` 导致对参数编码处理错误，请求不到正确的 URL。这个接口需要填写 `URLEncoding.queryString` 才能请求成功。

##### 进行网络请求
我在项目中创建了新文件 HttpTool 来对不同接口的数据进行处理。也导入了 SwiftyJSON 方便处理请求数据。

```
func getNetWorkRegion() {
    let provider = MoyaProvider<APIManager>()
    provider.rx.request(.getNetWorkRegion).subscribe { event in
        
        switch event {
        case .success(let response):
            K_CONFIGJSON.rg = JSON(response.data)["rg"].stringValue
        case .error(_):
            K_CONFIGJSON.rg = "1"      // 默认为 1---中国
        }
    }.disposed(by: bag)
}
```
##### 打印请求日志
[官方](https://github.com/Moya/Moya/blob/7c599570c2a60079dfefcc27fc67cb303feb48e1/Sources/Moya/Plugins/NetworkLoggerPlugin.swift)自带了一个插件来管理打印信息，但是这个插件打印信息太过全面，信息太多反而不好查看。我们可以通过[创建自定义插件](https://github.com/Moya/Moya/blob/7c599570c2a60079dfefcc27fc67cb303feb48e1/docs_CN/Examples/CustomPlugin.md)的方式来实现我们的需求。
创建一个类来遵循 PluginType 协议，并且实现协议方法。

```
final class HYNetWorkPlugin: PluginType {
    func willSend(_ request: RequestType, target: TargetType) {
        let req = request.request?.description ?? "无效的请求链接"
        print("++++++++++++++++\n\(req)\n++++++++++++++++")
    }
}
```
使用这个插件十分简单，只需要在创建的时候将插件对象作为参数传递到初始化方法中即可：

	let provider = MoyaProvider<APIManager>(plugins: [HYNetWorkPlugin()])

##### 参考资料

> [Moya 官方中文文档](https://github.com/Moya/Moya/blob/7c599570c2a60079dfefcc27fc67cb303feb48e1/docs_CN/README.md)
> [Swift - 网络抽象层库Moya的使用详解](http://www.hangge.com/blog/cache/detail_1797.html)