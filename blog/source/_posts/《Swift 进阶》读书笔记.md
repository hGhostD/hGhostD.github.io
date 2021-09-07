---
layout: 《Swift 进阶》读书笔记
title: 《Swift 进阶》读书笔记
date: 2017-12-18 17:26:04
tags: 
- Swift
- 读书
categories: 
- Swift
---
	Swift 既是一门高层级语言，又是一门低层级语言。Swift 是一门多范式语言。 

使用 Swift 也有一段时间了，这本书作为一门进阶用书写的十分优秀，翻译也十分浅显易懂。把学到内容作为笔记记录下来，也方便以后查看。
<!-- more -->
##### 第一章 介绍
这一章倒是并没有介绍太过深奥的内容，知识介绍了一些对 Swift 语言的整体印象，以及在使用过程中经常会遇到的术语，这些和其他编程语言都是通用的，就不一一列举出来了。就列出一些平常自己不知道的内容了。
             
> 在程序语言的论文里，`==` 有时候被称为结构相等，而 `===` 则被称为**指针相等**或者**引用相等**。

高阶函数：
> 如果一个函数接受别的函数作为参数，或者一个函数的返回值是函数，那么这样的函数就叫作**高阶函数**。

##### 第二章 内建集合类型
###### reduce
**运算符也是函数**
所以:
`let sum = fibs.reduce(0) { totle, num in totle + num}`
完全等价于
`let sum = fibs.reduce(0, +)`
reduce 的具体实现是:

```
extension Array {
    func reduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Element) -> Result) -> Result {
        var result = initialResult
        for x in self {
            result = try nextPartialResult(result, x)
        }
        return result
    }
}
```
###### flatMap
flatMap 和 map 类似，但是它们直接的区别是 faltMap 返回的是另一个数组，而不是单独的元素。
flatMap 的具体实现是:

```
extension Array {
    public func flatMap<SegmentOfResult : Sequence>(
        _ transform: (Element) throws -> SegmentOfResult
        ) rethrows -> [SegmentOfResult.Element] {
        var result: [SegmentOfResult.Element] = []
        for element in self {
            result.append(contentsOf: try transform(element))
        }
        return result
    }
}
```
###### 常用的函数参数
* map 和 flatMap 对元素进行变换。
* filter 元素是否应该被包含在结果中。
* reduce 将元素河滨到一个总和的值中。
* sequence 序列中下一个元素。
* forEach 对每一个元素进行操作。
* sort 对元素进行排序。
* index, contains 元素符合某个条件。
* min max 元素中最小值和最大值。
* starts 元素是否相等。
* split 切割元素。

##### 第三章 集合类型协议

###### Sequence
Sequence 协议是集合类型结构中的基础。一个序列（Sequence）代表的是一系列具有相同类型的值，你可以对这些值进行迭代。遍历一个序列最简单的方式是使用 for 循环。
这个函数有两个版本：
第一种方法：`sequence(first: next:)`。将使用第一个参数的值作为序列的首个元素，并使用 `next` 参数传入的闭包生成序列的后续元素。这里先定义一个 next 的闭包，用来生成一个随机数且要比前一个数小，到零为止:

```
let calculte = { (previouse: UInt32) -> UInt32? in
    let newValue = arc4random_uniform(previouse)
    guard newValue > 0 else {return nil}
    return newValue
}
```
定义好这个 next 闭包，就能够直接构建好一个 Sequence。`let randomNumbsers = sequence(first: 100, next: calculte)`。其中定义好初始化数据 first 为 100。
`Array(randomNumbsers) // [100, 56, 9, 6, 3, 2]`
第二个方法：`sequence(state: next:)`。因为它可以在两次 next 闭包被调用之间保存任意的可变状态，所以它更强大一些。这里我们通过它来创建一个[斐波那契序列](https://zh.wikipedia.org/wiki/%E6%96%90%E6%B3%A2%E9%82%A3%E5%A5%91%E6%95%B0%E5%88%97):

```
let calculte2 = { (state: inout (Int, Int)) -> Int? in
    let result = state.0
    state = (state.1, state.0 + state.1)
    return result
}
```
定义初始数据为一个元组 (0, 1)，再将这个元素传递给 next，然后取出元组中第一个数据，并修改元组数据:第一个数据 -> 第二个数据，第二个数据 -> 第一个与第二个数据的和。
`fibs = sequence(state: (0, 1), next: calculte2)`。
`Array(fibs.prefix(10)) // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]`
> `sequence(first:next:)` 和 `sequence(state:next:)` 的返回值类型是 `UnfoldSequence`。这个术语来自函数式编程，在函数式编程中，这种操作为成为展开(unfold)。sequence 是和 reduce 对应的(在函数式编程中 reduce 有常被叫做 fold)。reduce 将一个序列缩减(或者说折叠)为一个单一的返回值，而 sequence 则将一个单一的值展开形成一个序列。

##### 第四章 可选值
###### if let
使用 `if let` 来进行可选绑定(optional binding)要比使用 `switch` 语句稍好一些。

```
let urlString = "http://www.objc.io/logo.png"
if let url = URL(string: urlString),
	let data = try? Data(contentsOf: url),
	let image = UIImage(data: data) 
{	
	let view = UIImageView(image: image)
	PlaygroundPage.current.liveView = view
}
```

###### while let
`while let ` 和 `if let` 非常相似，它代表一个当遇到 nil 时终止的循环。注意，一旦条件为 false，循环就会停止(也许你错误地认为 where 条件会像 filter 那样工作，其实不然)。

###### nil 合并运算
`??` 和 Objective-C 中的 `?:` 十分相似，但是它们还是有不同的地方，那就是 Swift 中的可选值**不是指针**。 
我们可以对 Array 进行扩展来包含检查索引值是否在数组边界内:

```
extension Array {
    subscript(safe idx: Int) -> Element? {
        return idx < endIndex ? self[idx] : nil
    }
}
// 现在就可以这样写
array[safe: 5] ?? 0 // 0
```
合并操作也能够进行链接----如果有多个可能的可选值，并且想要选择第一个非 nil 的值，则可以将他们按顺序合并:

```
let i: Int? = nil
let j: Int? = nil
let k: Int? = 42
i ?? j ?? k // Optional(42)

```

##### 第五章 结构体和类
###### 值类型
结构体(和枚举)是**值类型**，而类是**引用类型**。因为结构体只有一个持有者，所以它不可能造成引用循环。而对于类和函数这样的引用类型，我们需要特别小心，避免造成引用循环的问题。
如果想要将两个 Point 相加，则可以对 + 操作符进行重载。在方法中，我们可以将成员相加，并返回新的 Point:

```
func +(lhs: Point, rhs: Point) -> Point {
	return Point(x: lhs.x + rhs.x, y: lhs.y + rhs.y)
}
screen.origin + Point(x: 10, y: 10) // (x: 20, y: 20)
```
如果想要改变 self，或者嵌套的(比如 self.origin.x)任何属性，就需要将方法标记为 `mutating`。只有使用了这个关键字，我们才能在方法内部进行改变。
Swift 的结构体一般被存储在栈上，而非堆上。如果结构体太大，它也会被存储在堆上。
###### 捕获列表
为了打破循环引用，需要保证闭包不去引用视图。通过使用**捕获列表**并将捕获变量 view 标记为 weak 或者 unowned 来达到这个目的。

```
window?onRotate = { [weak view] in
	print("We now also need to update the view: \(view)")
}
```

##### 第六章 函数

1. 函数可以像 Int 或者 String 那样被赋值给变量，也可以作为另一个函数的输入参数，或者另一个函数的返回值来使用。
2. 函数能够**捕获**存在于其作用域之外的变量。
3. 有两种方法可以创建函数，一种是使用 func 关键字，另一种是 { }。在 Swift 中，后一种被称为闭包表达式。

##### 第七章 字符串
这一章主要介绍了 String，但是书中还是介绍 Swift3.0 的版本，而最新的 Swift4.0 对此更改了很多。我就直接记录一些 String 的变化吧。
String 现在已经更改为一个集合类型了，所以现在直接废除了 `.characters` 这个属性了。其他的操作也都遵循了 Collection 协议了。

##### 第八章 错误处理
###### 抛出和捕获
Swift 没有使用返回 Result 的方法来表示失败，而是将方法标记为 throws。Result 是作用于类型上的，而 throw 是作用于函数的。

如果我们想要在错误中给出更多的信息，则可以使用带有关联值的枚举。(任何遵循 Error 协议的类型都可以被抛出函数作为错误抛出。)

```
enum LoginError: Error {
    case wrongUser
    case passwordError(user: String)
}
```
> 你可以指定你的函数可能抛出的具体错误类型，但这并不是必须的。因为现在 Swift 中的**错误类型是无类型的**，所以通过文档来说明你的函数会抛出怎样的错误是非常重要的。

```
func login(user: String, passWord: String) throws {
    if user.isEmpty {
        throw LoginError.wrongUser
    }
    
    if passWord != "123" {
        throw LoginError.passwordError(user: user)
    }
    print("登录成功！！！")
}

do {
    try login(user: "hu", passWord: "111")
}catch LoginError.wrongUser {
    print("用户不存在")
}catch LoginError.passwordError(let id) {
    print("\(id)的密码输入错误！！！") // hu的密码输入错误！！！
}catch {
    print(error.localizedDescription)
}
```
可以看出，在 Swift 中，我们虽然把这块内容叫做“异常”，但是实质上它更多的还是“错误”而非真正意义上的异常。

###### 使用 defer 进行清理
Swift 中的 `defer` ，围绕的代码块一定会在函数返回时被执行（就是相当于在 return 之后再执行一些操作）。
`defer` 类似于其他语言的 `finally`，但是 `defer` 不只是用于错误处理，可以将 `defer` 放在代码块的任意地方。
如果同一个作用域里使用多个 `defer`，那么它们会被逆序执行，可以把它想象成一个栈。用意在于，比如我们开启数据库然后连接，用 `defer` 就能自然的先关闭连接，再关闭数据库。

###### 高阶函数和错误
刚才我们就已经看出来了，在实际使用 Swift 中，经常能遇到网络请求这种异步处理错误的情况，如果按照刚才的写法，就是使用 `throws` 来捕获异常进行处理，这并不是一个好的方法，通常也不建议这么使用，因为可选值和 `Result` 作用于类型，而 `throws` 只对函数类型起效。将一个函数标注为 throws 意味着这个函数可能会失败。所以为了解决异步处理产生的问题，默认的规则就是定义 `Result<T>`：

```
enum Result<T> {
    case success(T)
    case fail(LoginError)
}
```
Result 是异步错误处理的正确道路。不好的地方在于，如果你已经在同步函数中使用 throws 了，那么再在异步函数中转为使用 Result 将会在两种接口之间导入差异。

##### 第九章 泛型
###### 使用闭包对行为进行参数化
对 Array 进行判断，标准库中提供的 `contains` 是这么做的:

```
extension Sequence {
	/// 根据序列是否包含满足给定断言的元素，返回一个布尔值。
	func contains(where predicate: (Iterator.Element) throws -> Bool)
		return -> Bool
}
```
也就是说，它接受一个函数，这个函数从序列中取出一个元素，并对他进行一些检查。

```
let isEven = { $0 % 2 == 0}
(0...5).contains(where: isEven)  // ture
[1,3,99].contains(where: isEven) //false
```

###### 使用泛型进行代码设计
让我们来写一些与网络服务交互的函数。比如获取用户列表的数据，并解析为 User 数据类型。

```
func loadUsers(callback: ([User]?) -> ()) {
	let usersURL = webserviceURL.appendingPathComponent("/users")
	let data = try? Data(contentsOf: usersURL)
	let json = data.flatMap {
		try? JSONSerialization.jsonObject(with: $0, options: [])
	}
	let users = (json as? [Any]).flatMap { jsonObject in
		jsonObject.flatMap(User.init)
	}
	callback(users)
}
```
这个函数会发生三种错误情况：URL 加载可能失败，JOSN解析可能失败，通过 JSON 数组构建用户对象也可能失败。

现在，如果我们想要写一个相同的函数来加载其他资源。比如，我们需要一个加载博客文章的函数，它看起来是这样的：

`func loadBlogPosts(callback: ([BlogPost])? -> ())`
此函数的实现和前面的用户函数几乎相同。相比于复制和粘贴，将函数中 User 相关的部分提取出来，将其他部分进行重用，会是更好的方式。
##### 第十章 协议
###### 面向协议编程和协议拓展
假设我们有一个只有 String 属性的 结构体 Obj

```
struct Obj {
    var value = "0"
    mutating func add(str: String) {
        let num = Int(value) ?? 0 + 1
        value = String(num)
    }
}
```
现在我们想通过协议的方式对一个 String 进行加一操作：

```
protocol transform {
    mutating func addOne(str: String)
}
```

只要对结构体 Obj 进拓展并具体实现加一方法，来满足协议:

```
extension Obj: transform {
    mutating func addOne(str: String) {
        let n1 = Int(str) ?? 0
        let n2 = Int(value) ?? 0
        let tmp = n1 + n2
        value = String(tmp)
    }
}
```
现在只需要假设 content 是满足协议的，就可以实现独立的方法：

```
var content: transform = Obj(value: "1")
content.addOne(str: "2") 		// Obj(value: "3")
```
除此之外，同样可以通过协议拓展的方式。
###### 协议内幕
当我们通过协议类型创建一个变量时，这个变量会被包装到一个叫存在容器的盒子中。
使用泛型参数确实要比使用协议类型高效的多。通过使用泛型参数，可以避免隐式的泛型封装。

```
// 隐式打包
func printProtocol(array: [CustomStringConvertible]) {
	print(array)
}
// 没有打包
func printGeneric<A: CustomStringConvertible>(array: [A]) {
	print(array)
}
```