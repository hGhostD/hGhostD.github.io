---
layout: swift
title: Swift的正则表达式
date: 2017-11-03 15:36:21
tags: 
- Swift
categories: 
- Swift
---
有的时候会遇到了需要自定义正则来判断输入信息  
<!-- more -->

```
extension String {
    func regularExpressions() -> Bool {
        if self.isEmpty {
            return false
        }
        //简单判断邮箱或者是手机号
        let photoReges = "^([A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,4})
        ||1[0-9]{10}$"
        
        let test = NSPredicate(format: "SELF MATCHES %@", photoReges)
        return test.evaluate(with: self)
    }
}
```
这里为了省事，直接给 String 添加了拓展方法。以后可以根据需求添加正则规则。
<br>
>参考资料
>[Swift 正则表达式规则](http://www.jianshu.com/p/d569dc998073)