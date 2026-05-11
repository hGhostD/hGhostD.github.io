---
layout: swift
title: Swift实现可编辑标签页
date: 2018-03-22 14:19:46
tags: 
- Swift
categories: 
- Swift
---
工作中遇到一个需要自定义的标签页，在网上找到了一个比较合适的 demo，可惜是以 Objective-C 实现的。就正好自己再用 swift 重写一下，也学习一下别人的思路。
<!-- more -->
#### SDTagsView  
[这里](https://github.com/SlowDony/SDTagsView) 是原作者的 demo，他分别采用了 Label 和 CollectionView 两种实现方式。不过作者比较推荐第二种，我也只以 CollectionView 的方式实现。
![SDTagsView](https://github.com/SlowDony/SDTagsView/blob/master/SDTagsView/SDEditTagsView.gif?raw=true)
#### 目录结构
因为我们这个 UI 控件可能需要在任意的 UIViewcontroller 中点击出现。所以我采用将这个控件同样以一个 UIViewcontroller 为承载。在需要的时候调用 present 弹出即可。目录结构如下:
![](https://github.com/hGhostD/MarkDownPhotos/blob/master/Swift%20%E7%AC%94%E8%AE%B0/%E6%A0%87%E7%AD%BE%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png?raw=true)
#### 实现 HUEditTagsViewController
TagsViewController 作为一个 Controller，是放置 view 和相关数据的基本控件。我们需要在此定义好数据源和相关的 view。

```
var myTagsArr: [String] = []
    var moreTagsArr: [String] = []
    private var dataArr: [[String]] {
        return [self.myTagsArr, self.moreTagsArr]
    }
    typealias tagsBlock = ([String], [String]) -> Void
    var saveBlock: tagsBlock?
    
    var tagsView : HUCollectionTagsView?
```
定义好必要的属性后，在 viewDidLoad 方法中做好相关的设置就可以了。这里我定义了3个数组，第一个数组放置我的标签内容，第二个数组放置更多标签的内容，第三个数组只有 get 方法，是将前两个数组放置到一起，传递给 view 作为数据源使用。
#### 实现 HUCollectionTagsView
这个作为一个 collectionview，实现它并不复杂。需要注意的是，我们这个 collectionview 实现头部view，也同样需要注册 header 的 cellIdefntifier，否则会造成崩溃。

```
self.register(UICollectionReusableView.self, forSupplementaryViewOfKind: UICollectionElementKindSectionHeader, withReuseIdentifier: cellHeaderIdentifier)
```
。需要在初始化的时候注册，和 cell 一样。
#### 实现 HUCollectViewCell 和 HUCollectionTagsFlowLayout
这两个配置文件是对 collectionview 的具体必要配置，可以按照自己的需求进行自定义。
#### 点击移动 item
至此，整体的结构已经实现完成，最后要实现一些点击事件的交互逻辑就可以完成。

由于点击事件的代理方法是绑定在 view 中的，但是我们想要点击时对 Controller 中的数据进行修改，就需要将事件传递出去。实现方式有很多，可以使用代理、block等。这里我们采用代理方式，需要自定义一个协议。

```
protocol HUCollectionTagsViewDelegate {
    func collectionTagsView(collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath)
}
/// HUCollectionTagsView 
var hu_delegate: HUCollectionTagsViewDelegate?

func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
	self.hu_delegate?.collectionTagsView(collectionView: self, didSelectItemAt: indexPath)
}
```
这个协议只有一个必要实现的方法，就是响应点击具体的 item。 
再由 HUEditTagsViewController 来实现：

```
func collectionTagsView(collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
		/// 处理点击 改变数据内容
}
```
做的时候觉得数据的传递绑定处理的不太完善，但是一直也没想到更好的处理办法，还需要进行优化，如果后面想到了，再来修改吧。
最后实现效果：
![](https://github.com/hGhostD/MarkDownPhotos/blob/master/Swift%20%E7%AC%94%E8%AE%B0/%E6%A0%87%E7%AD%BE%E6%95%88%E6%9E%9C.gif?raw=true)
完整代码在[这里](https://github.com/hGhostD/HUTagsView)

<br>
<br>
> 参考资料
> [SDTagsView1.0](https://github.com/SlowDony/SDTagsView)