---
layout: 《函数式 Swift》读书笔记
title: 《函数式 Swift》读书笔记
date: 2018-01-10 16:03:21
tags: 
- Swift
- 读书
categories: 
- Swift
---
		函数在 Swift 中是一等公民，换句话说，函数可以作为参数被传递到其他函数，
		也可以作为其他函数的返回值。函数式编程的核心理念就是函数是值。
<!-- more -->
#### 函数式思想
在 Swift 中计算和传递函数的方式与整形或布尔型没有任何不同。
假设有一个 struct 包含 x 和 y 坐标:

```
struct Position {
    let x: Int
    let y: Int
}
```

此时，定义一个 typealias，表示把 Position 转化为 bool 的函数。 
`typealias Region = (Position) -> Bool`。

接下来创建函数的返回值就可以是一个函数(Region) 

```
func circle(radius: Int) -> Region {
    return { loc in return loc.x < radius }
}
```

其中 

```
{ (参数) -> (返回值) in 
	(函数具体的操作)
	return (返回值)
}
```
大括号内就是表示一个函数。
 
#### 案例研究：封装 Core Image
书中的用例是对 CIImage 进行封装。为了让自己更明白本章所讲的内容，我把 CIImage 换成了 Int 类型来实现，两者的本质并无太大的区别，重点是对函数化这一概念要有清晰的认识。
我们首先将 Filter 定义为一个函数，该函数接受 Int 型作为参数，并且返回一个新的 Int 型。`typealias Filter = (Int) -> Int`。
首先，我们定义一个简单的 +1 的方法，返回值是 Filter(**是一个函数**)。 

```
func addOne(describe: String) -> Filter {
    return { number in
        return number + 1
    }
}
```
这样调用起来就是： `addOne(describe: "简单的 10+1")(10) // 11` 
接下来在的定义一个 +2 的方法，同上

```
func addTwo(describe: String) -> Filter {
    return { number in
        return number + 2
    }
}
```
到这里都挺简单的，现在如果我们要再实现一个 +3 的方法，再复制就显得太蠢了，所以转换一下思路，通过 +1 和 +2 的方法实现：

```
func addThree(describe: String) -> Filter {
    return { number in
        let one = addOne(describe: "+ 1:")(number)
        return addTwo(describe: "+ 2:")(one)
    }
}
addThree(describe: "+ 3:")(100) // 103
```
##### 复合函数
现在，如果我们想要实现 0 + 1 + 2 + 3 的功能，就可以这么调用：
`addThree(describe: "3")(addTwo(describe: "2")(addOne(describe: "1")(0))) // 6`。
感觉这么调用并不友好，我们需要优化一下实现方式。书中给了一种方案是使用**复合函数**。
首先定义一个用于组合的函数：

```
func compose(filter1: @escaping Filter, fileter2: @escaping Filter) -> Filter {
    return { num in fileter2(filter1(num)) }
}
```
这样可以使用复合函数来定义：
`compose(compose(addOne(describe: "1"), addTwo(describe: "2")), addThree(describe: "3"))(100) // 106`
##### 自定义运算符
但是看起来也很不容易理解。为了让代码更具可读性，我们可以再进一步，为组合引入运算符。

```
infix operator >>>: pro
precedencegroup pro {
    associativity: left
}

func >>>(filter1: @escaping Filter, fileter2: @escaping Filter) -> Filter {
    return { num in fileter2(filter1(num)) }
}
```
这样就可以用运算符来实现了，

```
let result = addOne(describe: "1") >>> addTwo(describe: "2") >>> addThree(describe: "3")
result(0) // 6
```
##### 柯里化
函数接受参数之后，返回一个**闭包**，然后等待第二个参数。`add(1)(2)`。函数中的箭头 -> 向右结合。这也就是说没你可以将 A -> B -> C 理解为 A -> (B -> C)。

#### Map、Filter 和 Reduce
##### Map 和 Filter
关于 Map 官网的描述是:
> Returns an array containing the results of mapping the given closure over the sequence’s elements.

翻译过来就是: 返回一个数组，数组中的元素是由闭包提供的映射序列产生的。

`func map<T>(_ transform: (Element) throws -> T) rethrows -> [T]`
其中关于参数的描述是:
> A mapping closure. transform accepts an element of this sequence as its parameter and returns a transformed value of the same or of a different type.

一个映射的闭包，以一个序列的**每个元素**作为参数，返回一个相同或不同的类型**新值**。可以知道我们需要提供的参数是**一个闭包**。这个闭包中的参数和返回值可以是任意类型的（由泛型定义）。

Map的定义是这样的：

```
extension Array {
	func map<T>(transform: Element -> T) -> [T] {
		var result: [T] = []
		for x in self {
			result.append(transform(x))
		}
		return result
	}
}
```
所以在调用的时候，我们就需要提供一个变换方法的闭包。

```
array.map { $0.lowercaseString } // ["vivien", "marlon", "kim"]
```

##### Reduce 
关于 Reduce 官网的描述是：
> Returns the result of combining the elements of the sequence using the given closure.

翻译过来是：返回组合运算的结果，组合运算是由提供的序列经过闭包处理每个元素产生的。

`func reduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Element) throws -> Result) rethrows -> Result`

##### 泛型和 Any 类型
除了泛型，Swift 还支持 Any 类型，它能代表任何类型的值。泛型可以用于定义灵活的函数，类型检查仍然由编译器负责；而 Any 类型则可以避开 Swift 的类型系统（**所以应该尽可能避免使用**）。

泛型函数的类型十分丰富，这里我们可以考虑一下上一章对自定义函数组合运算符 >>> 的泛型实现:

```
infix operator >>|: pro
precedencegroup pro {
    associativity: left
}
func >>|<A, B, C>(f: @escaping (A) -> (B), g: @escaping (B) -> (C)) -> (A) -> (C) {
    return { x in g(f(x))}
}

let result = addOne(describe: "1") >>| addTwo(describe: "2") >>| addThree(describe: "3")
result(10) // 16
```

#### 可选值
`??` 运算符的简易定义是这样的：

```
infix operator ??

func ??<T>(lhs: T?, rhs: T) -> T {
    if let result = lhs {
        return result
    }else {
        return rhs
    }
}

let result: Int? = { return nil }()
result ?? 100		// 100
```
`??` 运算符提供了一个相比于强制可选解包更安全的替代，并且不像可选绑定一样繁琐。