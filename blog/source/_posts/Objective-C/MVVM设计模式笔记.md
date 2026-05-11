---
layout: swift
title: MVVM 设计模式笔记
date: 2017-11-14 14:52:35
tags: 
- Swift
categories: 
- Swift
---
很早就听说过 MVVM 系统模式，但是都是一知半解的。这次就借助泊学网视频重新搭建一个查看天气的项目，系统的研究一下 MVVM 设计模式。
<!-- more -->
#### 概述
简易的画了一个 MVVM 设计图。整体流程如下，接下来将根据这个图进行设计。
![MVVM](https://github.com/hGhostD/MarkDownPhotos/blob/master/MVVM/MVVM%E6%A6%82%E8%BF%B0.jpg?raw=true)
#### 准备工作
搭建好项目 Sky，并准备好图片。![](https://github.com/hGhostD/MarkDownPhotos/blob/master/MVVM/%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84.png?raw=true)
准备好需要的数据接口，这里使用了 [DarkSky](https://darksky.net/dev) 接口的。
#### UI搭建 （View）
平时的项目很少都不会使用 StroyBoard，所以借着这个机会使用一下。文字不太好描述，这里直接贴图了。![](https://github.com/hGhostD/MarkDownPhotos/blob/master/MVVM/StoryBoard.png?raw=true)
View 上的控件记得连接在 Controller 上！
#### Model创建
这里偷懒直接用 CuteBaby 生成了我们需要的 Model 格式。注意处理好 Model 中嵌套的 struct 名称，尽量不要重名，防止将自己弄混。
#### ViewModel创建
ViewModel 更多的是让 Controller 从数据处理中解放出来，所以在 Controller 文件夹中单独创建 ViewModels 文件夹，并且将相关代码文件放入其中。
ViewModel 负责处理 Model 中传过来的数据。通过重写 set 方法将 Model 获取，并写好其他属性的 get 方法将数据向外传递。

```
var weather: WeatherData! {
        didSet {
            if weather != nil {
                self.isWeatherReady = true
            }
            else {
                self.isWeatherReady = false
            }
        }
    }
var summary: String {
        return weather.currently.summary
    }

```
#### Controller搭建
整体的业务逻辑就可以在 Controller 上进行了，最为重要的部分就是将各个部分的连通，确保能够进行数据的传递。以下几点是比较关键的部分，注意不要忘记实现。
1. 让 Controller 持有 ViewModel 对象，并重写 Set 方法来刷新 UI.
2. 将 ViewModel 的数据传递给 View，保证 View 能够显示出 ViewModel 的内容。
3. 让 Controller 持有 View 控件的点击方法，并能够通过代理将方法传递给其他对象。

#### 处理数据逻辑、业务逻辑
现在四个基本的元素 M、V、VM 和 C 都已经准备完成。接下来就是关键的数据绑定了，像管道一样逐步将数据传递下去。开始调用方法请求数据就是管道的起点。在 Controller 中需要进行数据请求的地方实现下面的方法。

```
WeatherDataManager.shared.weatherDataAt(
            latitude: lat,
            longtitude: lon,
            completion: { response, error in
                if let error = error {
                    dump(error)
                }else if let response = response {
                    self.currentWeatherViewController.viewModel?.weather = response
                    self.weekWeatherViewController.viewModel = WeekWeatherViewModel(weatherData: response.daily.data)
                }
        })
```
整体的数据绑定思路如下:
1. 获取地理位置将参数传递给获取天气的方法，以此来获取数据 Model。
2. 通过回调获取到 response， 就是获取到 Model 传递给 ViewModel。
3. 让子页的 Controller 持有的 ViewModel 进行赋值。
4. 通过重写子页 Controller 中 ViewModel 的 Set 方法，调用 `updateView()` 方法，对子页 Controller 持有的 View 进行刷新赋值。注意：关于 UI 的刷新需要在主线程中进行。

至此，由 Controller -> Model -> ViewModel -> View 的单向数据绑定已经完成。
接下来就是实现修改 View 上的的点击时间实现更新 Model。就是从 View -> Controller -> ViewModel -> Model。
整体思路如下：
1. 由 View 控件获取点击事件，将点击方法传递(通过代理或者 Block )给 Controller。
2. 修改后的数据通过 Controller 持有的 ViewModel传递。
3. 修改 ViewModel 中保存的 Model。

关于 MVVM 简单的双向数据绑定就算完成了，更多的使用可以借助一下三方框架，如 RxSwift。