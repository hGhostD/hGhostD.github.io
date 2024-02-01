---
layout: Objective-c
title: 关于RestKit的使用
date: 2018-06-23 18:34:37
tags:
- Objective-C
categories:
- Objective-C
---
最新需要使用 RestKit 框架处理网络请求，查了一下资料发现网上的资料确实不多，但还是有人翻译了一些资料的，趁此机会学习一下吧。
<!-- more -->
#### 服务器搭建
使用 RestKit 框架，需要有一个支持 Restful 的网络接口，为了方便自己测试，我们可以自行搭建一个本地的 Restful 接口。方法不是很复杂可以参考 [搭建 Restful 服务器](http://mclspace.com/2016/06/26/node-json-server/)。搭建成功后访问本地 3000 端口，访问 http://localhost:3000/posts/1 可以获得返回 json。
#### RestKit 文档
关于使用 RestKit 的文章，可以参考[这里](https://segmentfault.com/a/1190000003745207)，十分详细我们可以参考这里的介绍来使用。
#### 集成 RestKit
还是推荐使用 cocoapods 集成，方便管理。
#### 创建解析 Model
RestKit 非常强大的功能就是能够对返回 json 进行映射处理。所以很多时候需要使用 `RKObjectMapping` 来管理映射对象。刚才已经成功搭建了自己的服务器，先访问一下看看 json 的格式，如下：

```
{
  "id": 1,
  "title": "json-server",
  "author": "typicode"
}
```
这个 json 很容易理解，我们只需创建一个 ResultModel 的 class，来映射 json 即可，只需要在 .h 文件中声明 key。
#### URL 请求设置
在发送网络请求前，一定要明确的信息就是 URL，在使用 RestKit 前，就需要创建好 URL 的使用类。

```
NSURL *baseUrl = [NSURL URLWithString:@"http://localhost:3000"];
AFRKHTTPClient *httpClient = [[AFRKHTTPClient alloc]initWithBaseURL:baseUrl];
    
RKObjectManager *objectManager = [[RKObjectManager alloc]initWithHTTPClient:httpClient];
[objectManager setRequestSerializationMIMEType:RKMIMETypeJSON];
httpClient.allowsInvalidSSLCertificate = YES;
```
相应的一些 header 的关键信息，就可以添加到 httpClient 中，例如 `[httpClient setDefaultHeader:@"Content-Encoding" value:@"gzip"];`。
#### RKResponseDescriptor 设置
在发送请求之前，还需要的关键一步就是对返回 response 的处理，在这里设置对返回 json 的映射处理。

```
RKObjectMapping *mapping = [RKObjectMapping mappingForClass:[ResultModel class]];
[mapping addAttributeMappingsFromDictionary:@{@"id":@"identifier",@"title":@"title",@"author":@"author"}];
NSIndexSet *statusCodes = RKStatusCodeIndexSetForClass(RKStatusCodeClassSuccessful);
RKResponseDescriptor *responseDescriptor = [RKResponseDescriptor responseDescriptorWithMapping:mapping 
method:RKRequestMethodGET
pathPattern:nil 
keyPath:nil
statusCodes:statusCodes];
[objectManager addResponseDescriptor:responseDescriptor];
```
#### 发送参数访问
最后一步就是想正确的路径发送参数进行访问了，很容易理解。

```
[RKObjectManager.sharedManager getObject:_manager path:@"posts/1"
parameters:nil
success:^(RKObjectRequestOperation *operation, RKMappingResult *mappingResult) {
	successBlock(mappingResult);
} failure:^(RKObjectRequestOperation *operation, NSError *error) {
	errorBlock(error);
}];                                      
```
至此，一个基本的访问请求已经可以实现了。但是 RestKit 支持的功能还有很多没有使用到，最近公司也有个需求是要使请求发送 Gzip 数据，这就需要对 RestKit 进行一些拓展了，以后会详细说明。

<br>
> [搭建 Restful 服务器](http://mclspace.com/2016/06/26/node-json-server/)
> [RestKit ,一个用于更好支持RESTful风格服务器接口的iOS库](https://segmentfault.com/a/1190000003745207)