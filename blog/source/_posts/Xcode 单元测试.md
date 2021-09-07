---
layout: swift
title: Xcode 单元测试
date: 2017-11-13 14:28:01
tags: 
- Swift
categories: 
- Swift
---
有的时候很需要进行单元测试来节省开发时间。之前一直不太熟悉这一块的知识，正好整理学习一下。
<!--- more --->
#### 模拟网络请求
这里以 Sky 这个项目作为演示。模拟获取天气的网络请求方法。关键代码如下:

```
enum DataManagerError: Error {
    case failedRequest
    case invalidResponse
    case unknown
}
struct WeatherData: Codable {
    let latitude: Double
    let longitude: Double
}

typealias CompletionHandler = (WeatherData?, DataManagerError?) -> Void
    
func weatherDataAt(latitude: Double, longtitude: Double, completion: @escaping CompletionHandler) {
        let url = baseURL.appendingPathComponent("\(latitude), \(longtitude)")
        var request = URLRequest(url: url)
        
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpMethod = "GET"
        
        self.urlSession.dataTask(with: request) { (data, response, error) in
            self.didFinishGettingWeatherData(data: data, response: response, error: error, completion: completion)
        }.resume()
    }
func didFinishGettingWeatherData(data: Data?, response: URLResponse?, error: Error?, completion: CompletionHandler) {
        if let _ = error {
            completion(nil, .failedRequest)
        }else if let data = data, let response = response as? HTTPURLResponse {
            if response.statusCode == 200 {
                do {
                    let weatherData = try JSONDecoder().decode(WeatherData.self, from: data)
                    completion(weatherData, nil)
                }catch {
                    completion(nil, .invalidResponse)
                }
            }else {
                completion(nil, .failedRequest)
            }
        }else {
            completion(nil, .unknown)
        }
    }
```
其中包括 3 个关键的可变对象： self.urlSession、request 和  completion。我们就需要单独对这 3 个数据进行模拟。
模拟结果分为 3 种：
1. 是否能获取到网络数据。(success)
2. 获取网络数据失败。 (failedRequest)
3. 无效的网络响应。 (invalidResponse)

#### 准备工作
创建好 Tests 文件。在此需要引入头文件来确保可以调用工程中的对象方法 `@testable import Sky`。在测试文件中，测试方法总是以 test 作为方法名的开头。
#### 测试能否获取网络数据
想要测试能否获取数据，self.urlSession 就不需要特意模拟，request 可以任意给经纬度进行拼接。最后获取 completion 中的 WeatherData,查看是否拥有数据。

```
func test_weatherDataAt_gets_data() {
        let expect = expectation(description: "Loading data form \(API.authenticatedUrl)")
        var data: WeatherData? = nil
        WeatherDataManager.shared.weatherDataAt(latitude: 52, longtitude: 100) { (response, error) in
            data = response
            expect.fulfill()
        }
        
        waitForExpectations(timeout: 5, handler: nil)
        XCTAssertNotNil(data)
    }
```
这里使用 `expectation` 来进行异步测试，当网络回调结果时候调用 `expect.fulfill()` 表示异步完成。否则调用 `waitForExpectations(timeout: 5, handler: nil)` 表明最多等待5秒。最后 `XCTAssertNotNil(data)` 表明如果 data 为 nil，则测试失败，进入断言。
#### 测试failedRequest
为了节省异步获取网络资源花费的时间，需要我们自己构造一个无效的网络响应。为了不影响项目的代码，我们需要自己定一个测试用的 URLSession。

```
class MockURLSession: URLSessionProtocol {
    var responseData: Data?
    var responseHeader: HTTPURLResponse?
    var responseError: Error?
    var sessionDataTask = MockURLSessionDataTask()
    
    func dataTask(with request: URLRequest, completionHandler: @escaping URLSessionProtocol.dataTaskHandler) -> URLSessionDataTaskProcol {
        completionHandler(responseData, responseHeader, responseError)
        return sessionDataTask
    }
}
```
通过构造 MockURLSession 中的请求 responseError 来进行模拟。

```
func test_weatherDataAt_handle_invalid_request() {
        let session = MockURLSession()
        session.responseError = NSError(domain: "Invalid Requset", code: 100, userInfo: nil)
        
        let manager = WeatherDataManager(baseURL: URL(string: "https://darksky.net")!, urlSession: session)
        
        var error: DataManagerError? = nil
        manager.weatherDataAt(latitude: 52, longtitude: 100) { (_, e) in
            error = e
        }
        
        XCTAssertEqual(error, DataManagerError.failedRequest)
    }
```
#### 测试invalidResponse
和测试 failedRequest 的思路一样，只不过这次我们构造的错误信息应该是状态码 200 而 data 数据错误的情况。

```
func test_weatherDataAt_handle_invalid_response() {
        session.responseHeader = HTTPURLResponse(url: URL(string: "https://darksky.net")!, statusCode: 200, httpVersion: nil, headerFields: nil)
        
        let data = "{".data(using: .utf8)!
        session.responseData = data
        
        let manager = WeatherDataManager(baseURL: URL(string: "https://darksky.net")!, urlSession: session)
        
        var error: DataManagerError? = nil
        
        manager.weatherDataAt(latitude: 52, longtitude: 100) { (_, e) in
            error = e
        }
        
        XCTAssertEqual(error, DataManagerError.invalidResponse)
    }
```
#### 性能测试
利用 XCTest 中的 `measure()` 方法进行性能测试。可以直观的观察出方法花费时间。

```
func test_performance() {
    self.measure {
     	// Time: 0.003 sec (100% better)
        for i in 1...100 {
           print(i)
        }
    }
}
```
#####  后记 —— 2017年12月26日
尝试在公司的项目上进行单元测试，没想到配置环境就遇到了好多问题，果然是对这个 XCode 工程的配置不是很熟悉啊。由于我的工程项目是以 Swift 为基础，加入 Objective-C 框架进行混编的，而且使用 CocoaPod 进行的配置，所以引入工程遇到了很多问题。幸好还是在网上找到了解决方案，这里就记录一下。
###### 导入 Pods
由于新增测试模块，所以在进行测试的使用也会用到 CocoaPod 导入的三方库，所以在 Podfile 文件也需要对测试模块进行配置。

```
platform :ios, '8.0'
use_frameworks!
def pods
    pod 'Alamofire', '~> 4.5.1'
    pod 'SwiftyJSON', '~> 4.0.0'
end
target 'HiDoc' do
    pods
end

target 'HiDocTests' do
    pods
end
```
重新 pod install 的时候回遇到某些文件找不到的情况。这是因为 Tests 其实也是一个工程，依旧需要重新配置 Build Settings 中的 Path。比对原来工程中的 Search Paths 添加到工程中。![](https://github.com/hGhostD/MarkDownPhotos/blob/master/XCode%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95/SearchPaths.png?raw=true)
别忘了还要修改 <font color=#d13f28>**User Header Search Paths**</font> !!!
有的时候依旧会报找不到文件，还需要修改 Precompile Prefix Header 为 YES。
基本的配置已经完成了，在 Tests 文件中引入 `@testable import HiDoc` 就能够进行引用。但是在实际情况中，我的工程总会报 `"CMPopTipView.h" file not found`。我查看了一下发现这个文件是由 Pod 导入的第三方库，是在 Objective-C 的 .h 文件中引用的，并没有放入桥接文件中。后来就将这个文件引用放到 .m 文件中就解决了。
###### 进行登录接口单元测试
使用测试账号进行登录，创建测试方法。

```
func test_login() {
        let email = "13600000002"
        let password = "123456"
        
        let expect = expectation(description: "login")
        var isSuccess = false
        var test = ""
        HYLoginTool.loginWithPromise(email: email, password: password).then { (result) -> (Void) in
            if (result["ret"].stringValue == "0") {
                isSuccess = true
            }else {
                isSuccess = false
            }
            test = result["error"].stringValue
            expect.fulfill()
            }.catch { (error) in
                isSuccess = false
                test = error.localizedDescription
                expect.fulfill()
        }
        waitForExpectations(timeout: 10, handler: nil)
        XCTAssert(isSuccess, test)
    }
```

#### 常用判断条件

```
XCTFail(format…) 生成一个失败的测试； 
XCTAssertNil(a1, format...)为空判断，a1为空时通过，反之不通过；
XCTAssertNotNil(a1, format…)不为空判断，a1不为空时通过，反之不通过；
XCTAssert(expression, format...)当expression求值为TRUE时通过；
XCTAssertTrue(expression, format...)当expression求值为TRUE时通过；
XCTAssertFalse(expression, format...)当expression求值为False时通过；
XCTAssertEqualObjects(a1, a2, format...)判断相等，[a1 isEqual:a2]值为TRUE时通过，其中一个不为空时，不通过；
XCTAssertNotEqualObjects(a1, a2, format...)判断不等，[a1 isEqual:a2]值为False时通过；
XCTAssertEqual(a1, a2, format...)判断相等（当a1和a2是 C语言标量、结构体或联合体时使用, 判断的是变量的地址，如果地址相同则返回TRUE，否则返回NO）；
XCTAssertNotEqual(a1, a2, format...)判断不等（当a1和a2是 C语言标量、结构体或联合体时使用）；
XCTAssertEqualWithAccuracy(a1, a2, accuracy, format...)判断相等，（double或float类型）提供一个误差范围，当在误差范围（+/-accuracy）以内相等时通过测试；
XCTAssertNotEqualWithAccuracy(a1, a2, accuracy, format...) 判断不等，（double或float类型）提供一个误差范围，当在误差范围以内不等时通过测试；
XCTAssertThrows(expression, format...)异常测试，当expression发生异常时通过；反之不通过；（很变态） XCTAssertThrowsSpecific(expression, specificException, format...) 异常测试，当expression发生specificException异常时通过；反之发生其他异常或不发生异常均不通过；
XCTAssertThrowsSpecificNamed(expression, specificException, exception_name, format...)异常测试，当expression发生具体异常、具体异常名称的异常时通过测试，反之不通过；
XCTAssertNoThrow(expression, format…)异常测试，当expression没有发生异常时通过测试；
XCTAssertNoThrowSpecific(expression, specificException, format...)异常测试，当expression没有发生具体异常、具体异常名称的异常时通过测试，反之不通过；
XCTAssertNoThrowSpecificNamed(expression, specificException, exception_name, format...)异常测试，当expression没有发生具体异常、具体异常名称的异常时通过测试，反之不通过
```
