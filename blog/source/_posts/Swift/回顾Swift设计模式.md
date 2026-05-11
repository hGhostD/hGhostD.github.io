---
layout: swift
title: 回顾Swift设计模式
date: 2017-11-07 10:22:22
tags: 
- Swift
categories: 
- Swift
---
《Swift 设计模式》已经看过一遍了，不过大多时候也是看完就忘记了，这里重新再看一遍并记录一下。
<!-- more -->
#### 对象模板模式
> 其实这个设计模式就是我们通常概念的 <font color=#d13f28>**面向对象**</font> 编程。使用 **类** 或者 **结构体** 作为数据类型及其逻辑的规范。创建对象时使用 **模板**，并在初始化是完成数据赋值。赋值时，要么使用模板中的 **默认值**，要么使用类或者结构体的 **构造器** 提供数值。
  
技巧：有时候可以通过添加计算属性来保护不希望被随意修改的属性
```
private var inValue = 0
  var outValue: Int {
      get {
          return inValue
      }
      set {
          inValue = max(0, newValue)
      }
  }
  
proClass.outValue = 10  // 10
proClass.outValue = -10 // 0
```
#### 原型模式

> 原型模式通过克隆已有的对象来创建新的对象，已有的对象即为原型

组件若想通过模板创建新的对象，必须掌握以下三个信息。
- 与所需创建的对象相关的模板
- 必须调用的初始化器
- 初始化器的参数名称和类型

> 将一个值赋给变量时，Swift会自动使用原型模式。值类型是使用结构体定义，而且所有Swift内置类型的底层都是用结构图实现。

换句话说: Swift 只会为 <font color=#d13f28>**值类型(结构体)**</font> 创建默认的<font color=#d13f28>**初始化器**</font>。而**不会**为 <font color=#d13f28>**引用类型(类)**</font> 创建<font color=#d13f28>**初始化器**</font>。这就会导致创建完原型对象后，之后再创建任意多个同类对象，使用 **结构体** 模板创建的对象不会带来内存开销问题，而使用 **类** 模板创建的对象会有内存指向的问题。(2个变量会指向同一个对象。)
为了解决这个问题可以实现 NSCopying 协议，来支持克隆对象，实现深复制。
```
class Appointment: NSObject, NSCopying {
	var name: String
	var day: String
	init(name: String, day: String) {
		self.name = name
		self.day  = day
	}
	// 在实现
	func copy(with zone: NSZone? = nil) -> Any {
        return AppointmentClass(name: self.name, day: self.day)
    }
}
var b1 = Appointment(name: "Bob",day: "1")
var w1 = b1.copy() as! Appointment
w1.name = "Ali"
w1.day  = "2"
// 此时使用 copy() 创建的 w1 此时就不会对 b1 产生印象
```
Tips: 只有类才可以实现 NSCopying 协议，结构体不可以。因为结构体本身实现了深复制。

#### 单例模式
> 单例模式能够确保某个类型的对象在应用中只存一个实例。

创建单例模式可以像 Objective-C 一样创建一个单线程进行初始化。但是在 Swift 有更好的方法。
```
final class Single: NSObject {
    public static let shared = Single()
    private override init() {}
    
    var title: String = ""
    var number: Int = 0
}
```
Tips:这里遇到一个问题，如果创建单例模式的类不引用 NSOject 继承的话，就会报错。
问题找到了，由于在 `init()` 方法引用了关键字 `override` 导致了单例类一定要继承 NSObject 才有 `init()` 方法，所以编译器报错，去掉 `override` 就可以了。
##### 并发处理
如果在多线程应用中使用单例，那就需要仔细考虑一个问题：当有不同的组件同时操作单例对象时，会产生哪些后果。
Tips: Swift 数组并不是线程安全的。也就是说两个或更多线程同时调用数组的 `append` 方法，会同一个数组进行操作，会破坏数据的结构。
> 注意：高效的并发编程需要谨慎的思考和丰富的经验。有时初衷也许是很好的，但是最终开发出来的应用可能会很慢，甚至出现卡死的现象。

操作 Swift 数组的内容不是一个线程安全的操作，单例对象在使用数组时需要并发保护。为了解决这个问题，可以使用 <font color=d13f28>**串行访问**</font> 确保同一时刻只允许一个 block 调用数组的 `append` 方法。
```
final class Single: NSObject {
    // 创建出一个线程用来对数组进行操作
    let array = [String]()
    private let arrayQ = DispatchQueue(label: "arrayQ")
    // 创建出一个方法专门对 array 进行操作
    func backup(item: String) {
        arrayQ.sync { self.array.append(item) }
    }
}    
```
Tips: Swift 3 以后使用这种方式调用 `dipatch_barrier`

```
let wirte = DispatchWorkItem(flags: .barrier) { 
    // write data
}
let dataQueue = DispatchQueue(label: "data", attributes: .concurrent)
dataQueue.async(execute: wirte)
```
#### 对象池模式
> 对象池模式是单例模式的一种变体，它可以为组件提供多个完全相同的对象，而非单个对象。对象池模式一般来管理一组可重用的对象，
 
 此模式解决的问题：需要限制某个类型对象的数量，但允许多个个对象的存在。例如图书管理系统，需要管理某本书，但是又是多本存在。常见情况:UITableviewCell 的重用

利用范式创造一个对象池

```
class Pool<T> {
    private var data = [T]()
    private let arrayQ = DispatchQueue(label: "arrayQ")
    ///创建GCD信号源
    private let semaphore: DispatchSemaphore
    
    init(items:[T]) {
        data.reserveCapacity(data.count)
        semaphore = DispatchSemaphore(value: items.count)

        items.forEach{
            data.append($0)
        }
    }
    
    func getFromPool() -> T? {
        var result: T?
        if semaphore.wait(timeout: .distantFuture) == .success {
            arrayQ.sync {
                result = self.data.remove(at: 0)
            }
        }
        return result
    }
    
    func returnToPool(item: T) {
        arrayQ.sync {
            self.data.append(item)
            semaphore.signal()
        }
    }
}
```
> 注释: reserveCapacity 这个方法非常有趣。数组类型的元素并不一定需要是连续的（除非你使用 ContiguousArray），但是它们大部分都是连续的。正因如此，我们常常分配一段内存空间，用来存储我们的数组——特别是当我们知道数组的大致大小时。例如，`map()` 方法总是会返回和调用该方法的序列一样大小的一个数组。所以，`map()` 方法在填充数组之前，应该会使用 `reserveCapacity`。这一点可能不容易理解。`map()` 作用于 SequenceType 类型，不仅仅是 CollectionType 类型。Sequence 类型并没有 count 属性——那么如何才能得到序列的长度呢？即使它有 count 属性，CollectionType 类型的该属性，应该返回 Index.Distance，而对于 `reserveCapacity` 来说，需要一个 Int 类型。

##### 使用信号量解决并发问题
创建一个信号量 `private let semaphore: DispatchSemaphore` 并且在初始化的时候初始化它。

```
class Pool<T> {
	init(items: [T]) {
		// 为信号源创建一个计数器
		semaphore = DispatchSemaphore(value: items.count)
	}
	func getFromPool() -> T? {
		// 每次调用 semaphore.wait 都会使计数器的值减一 
		if semaphore.wait(timeout: .distantFuture) == .success {
			//确保在分线程执行操作 如果再主线程操作 整个 APP 就会冻住
			arrayQ.sync {
			}
		}
	}
	func returnToPool(item: T) {
		arrayQ.sync {
			//
			semaphore.singal()
		}
	}
}
```

#### 工厂方法模式
> 工厂方法模式通过选取相关的实现类来满足调用组件的请求，调用组件无需了解这些实现类的细节以及它们之间的关系。当存在多个类共同实现一个协议或者共同继承一个基类时，就可以使用工厂方法模式。

实现工厂方法模式最简单的方式是定义一个全局函数。由于全局函数可以在整个应用内调用，因此调用组件可以方便地定位和调用全局函数。

```
func createRentalCar(passengers: Int) -> RentalCar? {
    var car: RentalCar?
    switch passengers {
    case 0...1:
        car = Sports()
    case 2...3:
        car = Compact()
    case 4...8:
        car = SUV()
    default:
        car = nil
    }
    return car
}
```

创建通用协议，使创建的工厂类遵循同一个协议。

```
protocol RentalCar {
    var name: String { get }
    var passengers: Int { get }
    var pricePerDay: Float { get }
}

class Compact: RentalCar {
    var name = "Golf"
    var passengers = 3
    var pricePerDay: Float = 20
}

class Sports: RentalCar {
    var name = "Sport"
    var passengers = 1
    var pricePerDay: Float = 100
}

class SUV: RentalCar {
    var name = "SUV"
    var passengers = 8
    var pricePerDay: Float = 75
}
```

最后，就能够在类中调用工厂方法创建对象，而不需要了解对象的详细情况。

```
class CarSelector {
    class func selectCar(passengers: Int) -> String? {
        return createRentalCar(passengers: passengers)?.name
   }
}
```

#### 建造者模式
> 使用建造者模式可以将创建对象所需的逻辑和默认配置值放入一个建造者类中，这样调用组件只需了解少量配置数据即可创建对象，并且无需了解创建对象所需的默认数据值。

建造者模式实际上在开发中很常用，是指很多时候都没有意识到自己已经使用了建造者模式。说白了就是在创建对象方法和创建对象之间添加了一个中间层。只需在创建对象的时候添加修改的参数，而中间层添加默认参数，从而创建出对象。Swift 中已经很好的能够通过给方法参数添加默认值来实现初始化方法。所以可以很轻松的使用建造者模式。例如：`init(name: String, age: Int = 24, sex: Bool = true)`。

#### 适配器模式
> 适配器模式通过引入适配器对两个组件进行适配的方式，可以让两个 API 不兼容的组件写作。

适配器模式通过对不同类的 API 进行适配，将应用使用的 API 映射到组件提供的 API 方式，使得两个不兼容的类可以相互协作。
 
实现适配器模式最优雅的方式是使用 Swift extension。使用 extension 可以为无法修改源码的类增加功能。

其实适配模式的精髓就是让不同的类都**遵循相同的协议**。让不同的类实现好同一个方法

#### 桥接模式
> 桥接模式通过分离应用的抽象部分与实现部分，使得他们可以独立的变化。为了更高的解决层级爆炸的问题,分离应用的抽象部分与实现部分,使他们可以独立的变化.

从表面上看,桥接模式与适配器模式甚是相似。毕竟桥接模式的功能就是充当依赖某个协议的类与另外协议之间的适配器。虽然桥接模式和适配器模式相似，但是它们的应用场景并不相同。当需要集成无法修改源码的组件时(比如第三方组件),可以使用适配器模式。当你能够修改组件源代码及其运行方式时，便可使用桥接模式。使用桥接模式不只是创建一个桥接类这么简单，还需要最组件代码进行重构，以分离通用的代码和平台相关代码。

假定现在有2个不同的协议，且协议拥有各自的实现方法。

```
protocol ClearMessageChannel {
    func send(message: String)
}

protocol SecureMessageChannel {
   func sendEncryptedMessage(encryptedText: String)
}
```
现在需要通过一个类来实现这2个协议的方法，就需要通过桥接的方式来实现。
创建一个桥接的类，类中属性遵守不同的协议，并为属性创建协议能够调用的方法

```
class Communicator {
    private let clearChannel: ClearMessageChannel
    private let secureChannel: SecureMessageChannel
    private let priorityChannel: PriorityMessageChannel

    init(clearChannel: ClearMessageChannel, secureChannel: SecureMessageChannel ) {
        self.secureChannel = secureChannel
        self.clearChannel = clearChannel
    }

    func sendCleartextMessage(mesage: String) {
        self.clearChannel.send(message: mesage)
    }

    func sendSecureMessage(message: String) {
        self.secureChannel.sendEncryptedMessage(encryptedText: message)
    }
}
```
接下里，需要创建出不同的类来遵循上述协议并实现协议方法

```
class Landline: ClearMessageChannel {
    func send(message: String) {
        print("Landline",message)
    }
}

class SecureLandLine: SecureMessageChannel {
    func sendEncryptedMessage(encryptedText: String) {
            print("Secure", encryptedText)
    }
}
```
最后,通过就能够通过创建桥接对象来实现协议方法。

```
var comms = Communicator(clearChannel: Landline(), secureChannel: SecureLandLine())

comms.sendCleartextMessage(mesage: "CLEAR")
comms.sendSecureMessage(message: "Sercure")
```

#### 装饰器模式

> 此模式在处理无法修改的类时能发挥强大的功能。可以在不修改对象所属的类或对象的使用者情况下，修改单个对象的行为。

为了实现装饰器模式，需要继承那个无法修改的类，以创建一个拥有该类所有方法和属性的类，这样才可以实现无缝替换原来的那个类。但是 **Swift 不建议使用继承**。尽量还是避免使用这种模式。

#### 组合模式

> 组合模式能够将对象以树形结构组织起来，使得外界对单个对象和组合对象的使用具有一致性。

组合模式的实现主要是通过让不同的 类 都**遵循统一的协议**，这样就能够实现一个管理类统一处理不同对象。
![](https://github.com/hGhostD/MarkDownPhotos/blob/master/Swift%20%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E7%BB%84%E5%90%88%E6%A8%A1%E5%BC%8F.jpg?raw=true)

#### 外观模式

> 外观模式可以简化复杂的常见任务 API的使用

#### 享元模式

> 享元模式可以在多个调用组件之间共享数据
 
享元模式不会修改外部数据，也不允许调用组件修改外部数据。这是享元模式非常重要的特征，允许对外部数据进行修改也是人们在实现享元模式时常犯的一个错误。

#### 代理模式

> 代理模式的核心是一个对象--代理对象，此对象可以用于代表其他资源，此对象可以用于代表其他资源。

#### 责任链模式

> 责任链模式负责组织管理一序列能够对调用组件的请求做出响应的对象。这里所说的对象序列被称为责任链，责任链中的每个对象都可能被用于处理某个请求。请求在这个链上传递，知道链上的某一个对象处理此请求，或者到达链的尾部。
 
当多个对象可以响应一个请求，而又不想将这些对象的细节暴露给调用组件时，便可以使用责任链模式。下面举例说明一下责任链模式。
创建一种消息类型 Message，包括发送者、接收者和消息内容:

```
struct Message {
    let from: String
    let to:   String
    let subject: String
}
```

现在需要创建一条责任链，作用是处理 Message，判断这条消息是内部消息(from 和 to 的邮箱是同一个)、外部消息(from 和 to 不是同一个邮箱)还是私密消息(subject 的内容是 Priority 开头)。

```
class Transmitter {
    var nextLinke: Transmitter?
    
    required init() { }
    
    func sendMessage(_ message: Message) {
        if (nextLinke != nil) {
            nextLinke?.sendMessage(message)
        } else {
            print("责任链到达底端，消息不再发送。")
        }
    }
    
    class func matchEmailSuffix(message: Message) -> Bool {
        // 判断收和发的邮箱是否是同一个
        if let index = message.from.index(of: "@"){
            let sub = String(message.from[index...])
            return message.to.hasSuffix(sub)
        }
        return false
    }
    // 创建一条责任链
    class func createChain() -> Transmitter? {
        let transmitterClasses: [Transmitter.Type] = [
            PriorityTransmitter.self,
            LocalTransmitter.self,
            RemoteTransmitter.self]
        
        var link: Transmitter?
        
        transmitterClasses.reversed().forEach {
            let existingLink = link
            link = $0.init()
            link?.nextLinke =  existingLink
        }
        return link
    }
}
```

接下来就需要将责任链中不同的责任类进行实现，他们都需要集成责任链的类，并重写 sendMessage 的方法，然后在 sendMessage 的方法中分别实现自己的需求。

```
class LocalTransmitter: Transmitter {
    override func sendMessage(_ message: Message)  {
        if (Transmitter.matchEmailSuffix(message: message)) {
            print("\(message.from) 发送的内部消息")
        } else {
            super.sendMessage(message)
        }
    }
}

class RemoteTransmitter: Transmitter {
    override func sendMessage(_ message: Message) {
        if (!Transmitter.matchEmailSuffix(message: message)) {
            print("\(message.from) 发送外部消息")
        } else {
            super.sendMessage(message)
        }
    }
}

class PriorityTransmitter: Transmitter {
    override func sendMessage(_ message: Message) {
        if (message.subject.hasPrefix("Priority")) {
            print("\(message.from) 发送 私人消息")
        } else {
            super.sendMessage(message)
        }
    }
}
```

最后在创建的责任链中注意实现 sendMessage 方法:

```
let message = [Message(from: "bob@163.com", to: "joe@qq.com", subject: "午饭吃啥？"),
               Message(from: "joe@qq.com", to: "ali@qq.com", subject: "你吃啥？"),
               Message(from: "pet@2", to: "all@2", subject: "Priority: 这是私密消息！")]

if let chain = Transmitter.createChain()  {
    message.forEach {
        chain.sendMessage($0)
    }
}
// bob@163.com 发送外部消息
// joe@qq.com 发送的内部消息
// pet@2 发送 私人消息
```

#### 命令模式
> 命令模式提供了一种封装方法调用的机制，基于这种机制我们可以实现延迟方法调用或替换调用该方法的组用。命令模式的核心是 **命令对象**。在其内部实现中，接受对象持有一个命令 **接收者对象** 的引用，并知道如何调用接收者的相关方法。接收者和调用指令是命令私有的，不应允许使用命令的调用组件去访问。命令对象唯一的可供公开访问的是 execution 方法。当调用组件需要执行相关命令时，直接调用 execution 即可。
