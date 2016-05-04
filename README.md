---
title: AsyncDisplayKit官方文档翻译
date: 2016-05-04 20:51:01
categories: 搬砖
tags: [渲染]
---

[AsyncDisplayKit 官方文档](http://asyncdisplaykit.org/docs/getting-started.html)

最近在拆解学习AsyncDisplayKit这个很知名的轮子，发现这个轮子内容还是非常庞大的，想要分解学习之前必然要先对这个项目如何使用如何工作有一个初步的概念，所以动手准备把官方文档简单的翻译一下，希望更多看不顺英文文档的人，能有个简单的粗略了解，有了这个粗略了解之后，打算再动手准备进行源码分析

完全没有翻译文档的经验，碰到一些用词不合适的时候，还是推荐对比这原文进行观看

如果大家愿意，可以一起帮忙修改，直接提pr
[AsyncDisplayKitDocTranslation Github地址](https://github.com/Awhisper/AsyncDisplayKitDocTranslation)

# 快速开始
## 准备开始

AsyncDisplayKit的基础单元是Node，ASDisplayNode是UIView的抽象，就好像UIView是CALayer的抽象，但是不同于Views只能在主线程使用，Nodes是线程安全的，你可以并行在后台线程，实例化他们，配置他们整体的层次结构等

iOS设备有一条黄金准则，想要保持用户交互的流畅和快速响应，你的app必须保证渲染达到每秒60帧。意思就是主线程只有1/60秒的时间来推动每一帧，执行所有布局和绘图代码的时间只有16毫秒！而且由于一些系统级别的开销，你的布局绘图代码一般情况超过10毫秒，就可能引起掉帧

AsyncDisplayKit可以让你把图形解码，文本计算，渲染，等其他UI开销的操作移出主线程，还有一些其他的小把戏，我们后面会降到

### Nodes节点
如果你之前使用过views，那么你应该已经知道如何使用nodes，大部分的方法都有一个等效的node，大部分的UIView和CALayer的属性都有类似的可用的。任何情况都会有一点点命名差异（例如，clipsToBounds和masksToBounds），node基本上都是默认使用UIView的名字，唯一的例外是node使用position而不是center

当然，你也可以直接访问底层view和layer，使用node.view和node.layer

一些AsyncDisplayKit核心节点包括：

- AsDisplayNode:UIView子类对应使用，用于自定义view。
- AsCellNode:UICollectionviewcell或UITableViewCell类对应使用，用于自定义cell。
- AsControlNode:UIControl类对应使用，用于自定义按钮。
- AsImageNode:UIImageView类对应使用，处理图形解码之类。
- AsTextNode:UITextView类对应使用，建造全功能的富文本支持库。

### Node Containers节点容器

当在项目中替换使用AsyncDisplayKit的时候，一个经常犯的错误就是把一个Node节点直接添加到一个现有的view视图层次结构里。这样做会导致你的节点在渲染的时候会闪烁一下

相反，你应该你应该把nodes节点，当做一个子节点添加到一个容器类里。这些容器类负责告诉所包含的节点，他们现在都是什么状态，以便于尽可能有效的加载数据与渲染。你可以把这些类当做UIKit和ASDK的整合点

下面有4种节点容器

- ASViewController. UIViewController 类型的容器，管理子节点
- ASCollectionNode. UICollectionView 类型的容器，管理使用ASCellNode
- ASTableNode. UITableView 类型的容器，管理使用ASCellNode
- ASPagerNode. 一种特殊的ASCollectionNode 可以当做 UIPageViewController类型

<!-- more -->
### 排版引擎
AsyncDisplayKit的排版引擎是非常强大并且独特的，基于CSS FlexBox模型。他提供了一种声明方式来约定自定义节点所属的子节点的大小和布局，当所有的节点同时被默认渲染和展现的时候，通过给每个节点提供一个ASLayoutSpec，异步的测量和布局。

这套排版引擎那个是基于ASLayouts的概念，他包含了位置，大小，ASLayoutSpecs等信息，ASLayoutSpecs定义了不同的布局概念用于计算输出ASLayout结果，排版区域最终由子节点和其他排版区域一同决定

其他排版区域包括：

- ASLayoutSpec. 为相关节点提供位置和大小
- ASStackLayoutSpec. 可以把节点按着类似stackview的方式排布
- ASBackgroundLayoutSpec. 设置节点背景.
- ASStaticLayoutSpec. 当要手动定义一组节点静态大小


## 收益
### 异步性能提升

AsyncDisplayKit是一个UI框架，最初诞生于Facebook App。最开始Facebook团队面临一个很核心的问题：怎么能保证主线程尽可能的简洁，AsyncDisplayKit就是答案。

现在，很多应用程序都会频繁使用手势以及物理动画，再不济，你的app也会很广泛使用某种形式的滚动试图，这类型的用户交互是完全决定于主线程的流畅，并且对主线程的负荷十分敏感，一个被阻塞的主线程就意味着掉帧，卡顿，意味着很不愉快的用户体验

AsyncDisplayKit的Node节点就是一个线程安全的抽象对象，基于UIView和CALayer

[图就不翻译了]

当使用node节点的时候，你可以直接访问大部分的view和layer的属性，唯一区别是，当使用正确的时候，nodes通过异步进行计算，布局，最后默认同时进行渲染

想看更多地异步性能提升，请查看example/ASDKgram app，这个工程对比了基于UIKit的社交媒体demo，和基于ASDK的社交媒体demo

在iPhone6+上的性能提升不是很明显，但是在4S上，性能差距非常之大

### 强大的用户体验

AsyncDisplayKit所带来的性能提升，可以让你为每个用户在所有设备上，在所有的网络环境下，提供强大的用户体验设计

### 强大的开发体验

AsyncDisplayKit一样也在追求开发人员的使用体验，追求iOS&tvOS跨平台的特性，追求swift&OC语言的兼容性。只需要很少的代码就能构建很棒的app，清晰的架构，健壮的代码（参见examples/ASDKgram这个例子）（开发这个的都是一些超级聪明工作3年多的工程师）

### 高级的开发工具

随着AsyncDisplayKit逐渐成熟，很多聪明的工程师都加入一起构建这个项目，可以大幅度节省作为开发者，使用ASDK的开发时间

先进技术

ASRunLoopQueue

ASRangeController 智能预加载

网络工具

automatic batch fetching (e.g. JSON payloads)自动批量抓取

## 安装

__CocoaPods安装__
AsyncDisplayKit可以使用cocoapods安装，将下面的代码添加进入`Podfile`

`pod 'AsyncDisplayKit'`

__Carthage安装__
AsyncDisplayKit可以使用Carthage安装，将下面的代码添加进入`Cartfile `

`github "facebook/AsyncDisplayKit"`

在终端执行`carthage update`来构建AsyncDisplayKit库，会自动在项目根目录下生成Carthage名字的文件夹，里面有个build文件夹，可以用来framework到你打算使用的项目中

__静态库__
AsyncDisplayKit可以当做静态库引入

1）拷贝整个工程到你的目录下，添加`AsyncDisplayKit.xcodeproj`到你的workspace

2）在build phases中，在Target Dependencies下添加AsyncDisplayKit Library

3）在build phases中，添加libAsyncDisplayKit.a, AssetsLibrary, Photos等框架到Link Binary With Libraries中

4）在build settings中，添加` -lc++`和`-ObjC`到 project linker flags

__引用AsyncDisplayKit__

```objectivec
#import <AsyncDisplayKit/AsyncDisplayKit.h>
```

# 核心概念

## 智能预加载
node的功能很强大的原因是具有异步渲染和计算的能力，另一个至关重要的因素是ASDK的智能预加载方案。

正如在`准备开始`提到的那样，使用一个在节点容器上下文之外的节点是非常的没有意义的，这是因为所有的节点都有一个当前界面状态的概念。

这个界面状态的属性是不断的通过ASRangeController来进行更新，ASRangeController是由容器创建和内部维护的。

一个节点在容器之外被使用，是不会使它的状态得到rangeController的更新。有时候会产生的闪屏现象是由于，一个节点刚刚被渲染到屏幕之后又进行了一次渲染

### 界面状态区域

当节点被添加到滚动或者分页的控件的时候，一般都会遇到下面的几种情况。这就是说scrollview正在滚动，他的界面属性会随着移动一直在更新。
每个节点一定会处于以下几种范围：

数据范围：距离显示最远的区域，他们的数据已经收集准备完毕，无论是通过本地磁盘还是API，但远没有进行显示处理
显示范围：在这个范围，显示任务，比如文本光栅化，图像解码等正在进行
可见范围：这个node，至少有一个像素已经被渲染到屏幕上

### ASRangeTuningParameters

每一个范围的大小都是全屏去计算的，一般情况下，默认的尺寸就能很好的工作，但是你也可以通过设置滚动节点的区域类型参数，很简单的进行调整。

[图就不翻译了，红色是可见范围，橙色是显示范围，黄色是数据范围]

上图就是一款app的截图，用户可以向下滚动，正如你所看到的，用户滚动方向区域（领先方向）的大小，要比用户离开方向区域（尾随方向）的大小，大得多。如果用户滚动的方向发生了改变，这两个区域的大小会动态的切换，以保证内存的最佳使用。你只需要考虑定义领先方向和尾随方向的区域大小，而不必担心滚动方向的变化。

在这个截图中，你还可以看到智能的预加载是如何在多维度下工作的，你可以看到一个垂直的滚动容器，你可以看到虽然有些node还未在红色的设备屏幕中出现，但是他有一个范围控制器，也有处在黄色的数据范围的node，和橙色的显示范围的node

### 界面状态回调
随着用户的滚动，nodes会移动穿过这些区域，并做出适当的反应，读取数据，渲染，等等。你自己创建的node子类可以很容易的通过实现相应的回调来精细设计他们

可见范围回调

`- (void)visibilityDidChange:(BOOL)isVisible;`

显示范围回调

`- (void)displayWillStart`

`- (void)displayDidFinish`

数据范围回调

`- (void)fetchData`

`- (void)clearFetchedData`


最后，千万别忘记调用super

## 自定义子类

写一个node的子类，很像写一个UIView的子类，这有几条准则需要遵守，来确保你能够充分发挥这个框架的潜力，确保你的节点正常工作

### 基本重载方法

`-init`

通常情况下，你都会写一些init方法，包括，调用`[super init]`，然后设置一些自定义的属性。这里需要记住的尤其重要的一点，你写的（node的）init方法必须能被任何的队列和线程调用，所以尤其要注意的一点就是，在init方法里确保不要初始化任何UIKit的对象

`-didLoad`

这个方法很像UIViewController的`-viewDidLoad `，这个时间点支持视图已经被加载完毕，此时可以保证是主线程，因此，你可以在这个时机，设置UIKit相关对象

`layoutSpecThatFits`

这个方法就是用来建立布局规则对象，产生节点大小以及所有子节点大小的地方，你创建的布局规则对象is maleable up until the point that it is return in this method（不明白╮(╯_╰)╭），经过了这个时间点后，它就不可变了。尤其重要要记住的一点事，千万不要缓存布局规则对象，当你以后需要他的时候，请重新创建

`layout`

在这个方法里调用super之后，布局规则对象会把所有的子节点都计算并且定位好，所以这个时间点是你手动进行布局所有子view的时机。或许更有用的是，有时候你想手动布局，但并不太容易创建一个布局规则对象，或者有时候你不想等所有子节点布局完毕，而只是很简单的手动设置frame，如果是这样的话，就在这个方法里写

## 排版引擎