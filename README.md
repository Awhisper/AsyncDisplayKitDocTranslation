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

这个方法就是用来建立布局规则对象，产生节点大小以及所有子节点大小的地方，你创建的布局规则对象一直持续到这个方法返回的时间点，经过了这个时间点后，它就不可变了。尤其重要要记住的一点事，千万不要缓存布局规则对象，当你以后需要他的时候，请重新创建

`layout`

在这个方法里调用super之后，布局规则对象会把所有的子节点都计算并且定位好，所以这个时间点是你手动进行布局所有子view的时机。或许更有用的是，有时候你想手动布局，但并不太容易创建一个布局规则对象，或者有时候你不想等所有子节点布局完毕，而只是很简单的手动设置frame，如果是这样的话，就在这个方法里写

## 排版引擎

AsyncDisplayKit的排版引擎是基于CSS Box模型，它具有类似UIKit（自动布局）一样的特征，一旦你习惯它，就会发现这是他最有用的特点。只要有足够的联系，你就会越来越习惯通过创建布局声明来实现基础约束。


想要参与这个过程的主要方式就是通过在子类中实现`layoutSpecThatFits:`方法，在这里你声明和建立布局规则对象，返回最重的包含所有信息的对象


```objectivec
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize
{
  ASStackLayoutSpec *verticalStack = [ASStackLayoutSpec verticalStackLayoutSpec];
  verticalStack.direction          = ASStackLayoutDirectionVertical;
  verticalStack.spacing            = 4.0;
  [verticalStack setChildren:_commentNodes];

  return verticalStack;
}
```

尽管这个例子非常简单，但是它给了你一个思路，如何去使用布局规则对象，一个stack布局规则对象为例，定义了一种子节点们相邻的布局方式，确定了方向，间隔的定义，他看起来很像UIStackView，但是可以支持低版本iOS

### ASLayoutable

布局规则对象的孩子，可以是任何对象，只要他遵从`<ASLayoutable>`协议，所有的nodes和布局规则对象都遵从这个协议，就是说你的布局，可以用任何你想的对象来建立。

比如，你想要添加8像素的间隔给这个你刚刚建立好的stack

```objectivec
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize
{
  ASStackLayoutSpec *verticalStack = [ASStackLayoutSpec verticalStackLayoutSpec];
  verticalStack.direction        = ASStackLayoutDirectionVertical;
  verticalStack.spacing            = 4.0;
  [verticalStack setChildren:_commentNodes];
  
  UIEdgeInsets insets = UIEdgeInsetsMake(8, 8, 8, 8);
  ASInsetLayoutSpec *insetSpec = [ASInsetLayoutSpec insetLayoutSpecWithInsets:insets 
                                      child:verticalStack];
  
  return insetSpec;
}

```

你可以很容易就完成

当然，使用布局规则对象需要一些联系，可以看layout section部分

# 节点容器
## 容器总览
### 在节点容器中使用节点

强烈建议你通过节点容器，来使用AsyncDisplayKit的节点，AsyncDisplayKit提供了下面4种节点

- ASViewController 类似 UIKit's UIViewController
- ASCollectionNode 类似 UIKit's UICollectionView
- ASPagerNode 类似 UIKit's UIPageViewController
- ASTableNode 类似 UIKit's UITableView

例子demo在每个节点容器的文档中被高亮展示了

想要详细的从UIKit移植app到AsyncDisplayKit信息，请看移植指南

### 使用节点容器有什么好处

介电容器自动管理着子节点的智能预加载，所有的子节点都可以异步的执行布局计算，获取数据，解码，渲染。正因为此，推荐在节点容器内使用节点

注意：确实可以直接使用节点不使用节点容器，但他们只会在出现到屏幕上的时候展现一次，这样会导致性能退化，或者闪烁

## ASViewController
ASViewController是UIViewController的子类，并且添加了很多和ASDisplayNode层级相关的功能

ASViewController可以被当做UIViewController来使用，包括套在一个UINavigationController，UITabBarController，UISpitViewController里面

使用ASViewController的一大好处是节省内存，当ASViewController离开屏幕的时候会自动减少其子node的数据范围（前文提到）和显示范围（前文提到），这是大型app管理的关键

除此之外还有更多地功能，把它当做你app的基础ViewController的基类是一个不错的选择。

UIViewController提供他自己的view，ASViewController提供了他自己的node，是在`-initWithNode:`方法中创建的

你可以参考下面这个ASViewController的子类PhotoFeedNodeController，在ASDKgram sample app里面，你可以看到他是如何管理和使用一个列表节点的

这个列表节点就是在`-initWithNode:`方法中创建的

```objectivec
- (instancetype)init
{
  _tableNode = [[ASTableNode alloc] initWithStyle:UITableViewStylePlain];
  self = [super initWithNode:_tableNode];
  
  if (self) {
    _tableNode.dataSource = self;
    _tableNode.delegate = self;
  }
  
  return self;
}
```

如果你的app已经有了很复杂的viewcontroller层级，你最好把他们都改成继承自ASViewController，就是说，即便你不使用ASViewController的`-initWithNode:`方法，你只是把它当做传统的UIViewController来使用，当你一旦选择不同的使用方式，他就能给你节点管理方面的支持

## ASTableNode

ASTableNode等效于UIKit的UITableview，而且可以拿来替换UITableView

ASTableNode替换UITableView以下方法
`- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath`

替换成下面这两种方法之一

`- (ASCellNode *)tableView:(ASTableView *)tableView nodeForRowAtIndexPath:(NSIndexPath *)indexPath`

`- (ASCellNodeBlock)tableView:(ASTableView *)tableView nodeBlockForRowAtIndexPath:(NSIndexPath *)indexPath`

建议你使用nodeBlock的版本，以便这些节点可以同时进行准备和显示处理，所有的子节点都可以在后台线程进行初始化，所以一定要保持他们的代码是线程安全的。

这两个方法都需要返回一个ASCellNode或者ASCellNodeBlock。一个ASCellNodeBlock是用来在后台创建ASCellNode的，注意ASCellNode在ASTableNode，ASCollectionNode，ASPagerNode都有使用

请注意这两个方法都不需要重用机制！

### 用ASViewController替换UITableViewController

AsyncDisplayKit并没有提供一个类似UITableViewController的东西，你需要使用ASViewController初始化一个ASTableNode

继续可以参照ASViewController的子类PhotoFeedNodeController，在ASDKgram sample app里面

一个ASTableNode在ASViewController的`-initWithNode: `方法初始化
```objectivec
- (instancetype)init
{
    _tableNode = [[ASTableNode alloc] initWithStyle:UITableViewStylePlain];
    self = [super initWithNode:_tableNode];
    
    if (self) {
      _tableNode.dataSource = self;
      _tableNode.delegate = self;
    }
    
    return self;
}
```

### node block线程安全警告
保证node block 的代码一定要是线程安全的，一方面要保证块里面的数据对外面是可访问的，所以你不应该使用block内的索引

继续参考子类PhotoFeedNodeController，在ASDKgram sample app里面的例子，` -tableView:nodeBlockForRowAtIndexPath:`这个方法

这个例子可以看出来，如何在使用索引，访问photo的模型

```objectivec
- (ASCellNodeBlock)tableView:(ASTableView *)tableView nodeBlockForRowAtIndexPath:(NSIndexPath *)indexPath
{
    PhotoModel *photoModel = [_photoFeed objectAtIndex:indexPath.row];
    
    // this may be executed on a background thread - it is important to make sure it is thread safe
    ASCellNode *(^cellNodeBlock)() = ^ASCellNode *() {
        PhotoCellNode *cellNode = [[PhotoCellNode alloc] initWithPhoto:photoModel];
        cellNode.delegate = self;
        return cellNode;
    };
    
    return cellNodeBlock;
}
```

### 列表行高

一个很重要的事情就是，ASTableNode并不提供类似UITableview的`-tableView:heightForRowAtIndexPath:`方法

这是因为节点基于自己的约束来确定自己的高度，就是说你不再需要写代码来确定这个细节

一个node通过`-layoutSpecThatFits:`方法返回的布局规则确定了行高，所有的节点只要提供了约束大小，就有能力自己确定自己的尺寸

在默认的情况下，tableNode提供了Cell的尺寸约束范围，最小宽度和最低高度是0，最大宽度是tablenode的宽度，最大高度是MAX_FLOAT，这就是说，tablenode的cell，总是填满tablenode的全宽，他的高度是自己计算自适应的

如果你对一个ASCellNode调用了`setNeedsLayout`，他会自动的把布局传递，如果整体所需要的大小发生了变化，表会被告知要进行更新

和UIKit不同的时，你不需要调用reload，这样很节省了代码，可以看ASDKgram sample来看一个table的具体实现


## ASCollectionNode

ASCollectionNode就是类似UIKit的UICollectionView，可以拿来代替UICollectionView

ASCollectionNode替换UICollectionView的时候需要把下面这个方法

`- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath;`

替换成下面2个之一

`- (ASCellNode *)collectionView:(ASCollectionView *)collectionView nodeForItemAtIndexPath:(NSIndexPath *)indexPath`

`- (ASCellNodeBlock)collectionView:(ASCollectionView *)collectionView nodeBlockForItemAtIndexPath:(NSIndexPath *)indexPath`

同tablenode一样，推荐使用block的版本

正如前面说过的

- ASCollectionNodes不需要重用。
- nodeBlock的方法建议使用。
- 块内要求是线程安全的，这一点很重要。
- ASCellNodes可由ASTableNode，ASCollectionNode和ASPagerNode使用

### 用ASViewController替换UICollectionViewController

AsyncDisplayKit并没有提供类似UICollectionViewController的类，你还是需要使用ASViewController，初始化的时候创建一个ASCollectionNode，在`-initWithNode:`方法里

```objectivec
- (instancetype)init
{
  _flowLayout     = [[UICollectionViewFlowLayout alloc] init];
  _collectionNode = [[ASCollectionNode alloc] initWithCollectionViewLayout:_flowLayout];
  
  self = [super initWithNode:_collectionNode];
  if (self) {
    _flowLayout.minimumInteritemSpacing  = 1;
    _flowLayout.minimumLineSpacing       = 1;
  }
  
  return self;
}
```

ASTableNode,ASPagerNode都是这样工作的

### 访问ASCollectionView

如果你用过以前版本的ASDK，你会发现，为了方便ASCollectionNode，ASCollectionView已经被移除了，

ASCollectionView是UICollectionView的子类，仍然是通过ASCollectionNode内部来使用。虽然他不可以被直接创建，但是他可以通过ASCollectionNode的.view属性来访问，但是别忘了，一个节点的视图，只有在viewdidload 或者didload之后，才可以进行访问

下面的这个例子可以看出来，直接访问ASCollectionView

```objectivec 
- (void)viewDidLoad
{
  [super viewDidLoad];
  
  _collectionNode.view.asyncDelegate   = self;
  _collectionNode.view.asyncDataSource = self;
  _collectionNode.view.allowsSelection = NO;
  _collectionNode.view.backgroundColor = [UIColor whiteColor];
}

```

### CELL计算和布局

就像之前提到过的，ASCollectionNode和ASTableNode不需要计算他们的CellNode的高度

现在，无论采用哪种UICollectionViewLayout，cell可以根据约束的大小来自动布局自适应，

不久之后，会有一个类似ASTableNode的`constrainedSizeForRow:`方法，但是现在，如果你需要在collectionNode里约束一个cellNode，你需要包装处理一下约束规则对象

### 例子

最详细的ASCollectionNode例子就是CustomCollectionView，包括自定义的ASCollectionNode与UICollectionViewLayout.

更多地demo请看

- ASDKgram
- CatDealsCollectionView
- ASCollectionView


## ASPagerNode

ASPagerNode是ASCollectionNode的子类，他特别定制了UICollectionViewLayout

使用ASPagerNode可以让你开发类似UIKit中UIPageViewController的效果，ASPagerNode目前只支持在滚动停留到正确的位置，但还不支持循环滚动

最核心的数据源方法如下

`- (NSInteger)numberOfPagesInPagerNode:(ASPagerNode *)pagerNode`

`- (ASCellNode *)pagerNode:(ASPagerNode *)pagerNode nodeAtIndex:(NSInteger)index`

`- (ASCellNodeBlock)pagerNode:(ASPagerNode *)pagerNode nodeBlockAtIndex:(NSInteger)index`

后面这两种方法就像ASCollectionNode 和 ASTableNode一样需要返回ASCellNode 或者ASCellNodeBlock，用于创建可以在后台线程运行的ASCellNode


注意，这些方法都不要写重用逻辑，不像UIKit，这些方法在即将要显示的时候，是不会调用的

`-pagerNode:nodeAtIndex:`会在主线程被调用, `-pagerNode:nodeBlockAtIndex:`更推荐使用，因为所有的node的初始化方法都可能在背景线程和主线程中调用，所以一定要确保block中的代码线程安全

### nodeblock线程安全警告

保证node block 的代码一定要是线程安全的，一方面要保证块里面的数据对外面是可访问的，所以你不应该使用block内的索引

可以看下面的例子

```objectivec
- (ASCellNodeBlock)pagerNode:(ASPagerNode *)pagerNode nodeBlockAtIndex:(NSInteger)index
{
  PhotoModel *photoModel = _photoFeed[index];
  
  // this part can be executed on a background thread - it is important to make sure it is thread safe!
  ASCellNode *(^cellNodeBlock)() = ^ASCellNode *() {
    PhotoCellNode *cellNode = [[PhotoCellNode alloc] initWithPhoto:photoModel];
    return cellNode;
  };
  
  return cellNodeBlock;
}

```

### 优化使用ASViewController

一个很有效的方式是，直接返回ASViewController中初始化好的ASCellNode，所以还是推荐使用ASViewController

```objectivec
- (ASCellNode *)pagerNode:(ASPagerNode *)pagerNode nodeAtIndex:(NSInteger)index
{
    NSArray *animals = self.animals[index];
    
    ASCellNode *node = [[ASCellNode alloc] initWithViewControllerBlock:^{
        return [[AnimalTableNodeController alloc] initWithAnimals:animals];;
    } didLoadBlock:nil];
    
    node.preferredFrameSize = pagerNode.bounds.size;
    
    return node;
}
```

在这个例子这种，你可以看到节点通过`initWithViewControllerBlock `方法进行约束，为了正确的布局，还是需要提供一个希望的framesize

例子

- PagerNode
- VerticalWithinHorizontalScrolling

# 节点

## ASDisplayNode

### 最基础的节点
ASDisplayNode是最主要的UIView和CALayer的抽象对象，他初始化的时候拥有一个UIView，同时UIView在初始化的时候拥有一个CALayer

```objectivec
ASDisplayNode *node = [[ASDisplayNode alloc] init];
node.backgroundColor = [UIColor orangeColor];
node.bounds = CGRectMake(0, 0, 100, 100);

NSLog(@"Underlying view: %@", node.view);
```
Node和UIView具有一样的属性，所以使用起来非常像UIKit

无论是view还是layer的属性，都可以通过node进行访问

```objectivec
ASDisplayNode *node = [[ASDisplayNode alloc] init];
node.clipsToBounds = YES;                       // not .masksToBounds
node.borderColor = [UIColor blueColor];  //layer name when there is no UIView equivalent

NSLog(@"Backing layer: %@", node.layer);
```

你可以看到，默认命名是照着UIView的习惯，除非这个属性是UIView不具备的，你就像处理普通UIView一样，去访问底层的CALayer

当我们使用了节点容器，节点的属性会在背景线程被设置和使用，背后的view/layer会延迟懒加载生成约束，你不需要去担心跳入背景线程要注意什么，因为框架都处理好了，但是你也要了解都发生了什么

### 页面包装

某些情况下，需要初始化一个节点，提供一个view当做基础view。这些view需要一个block来处理之后被保存的view（有点绕。。我也没太懂。）

这些节点展示的步骤是同步的，这是因为节点只有在被包上一层_ASDisplayView后，才可以异步，现在他只是包在普通UIView上

```objectivec
ASDisplayNode *node = [ASDisplayNode alloc] initWithViewBlock:^{
    SomeView *view = [[SomeView alloc] init];
    return view;
}];
```

这样可以让你把一个UIView子类包装成一个ASDisplayNode子类

唯一要注意的时node使用position，不是center

## ASCellNode

ASCellNode 可能是最常用的节点子类了，他可以被用于ASTableNode和ASCollectionNode

就像其他子类继承自ASDisplayNode，ASCellNode更重要的是需要重写`-init`方法初始化 `-layoutSpecThatFit:`来布局和计算，如果需要，重写`-didLoad`来添加额外的手势或者额外的布局

如果你不喜欢继承，你也可以使用`-initWithView `和`-initWithViewController`方法来返回一个节点，他的内部view就是通过已经存在的view来创建的

## ASTextCellNode

敬请期待......

## ASControlNode

敬请期待......

点击区域可视化

只用一行代码，就可以轻松的把所有的ASControlNode的点击区域可视化，通过这个工具hit test slop debug tool.

## ASButtonNode

ASButtonNode是ASControlNode的子类，提供了简单button的功能，有多重状态，标签，图片，和布局选项，开启layerBacking可以显著减少button对主线程的影响

功能：

- 支持更换背景图片
- 支持更换标签
- 支持更换文字对齐
- 支持更换内容边界控制
- 提供方法来制作富文本标签

小心：
选择selected属性的逻辑应该由开发者处理，点击buttonNode不会自动的开启selected

## ASTextNode

敬请期待......

## ASEditableTextNode

ASEditableTextNode提供了一个灵活的高效的动画有好的可编辑文本控件

功能：sizeRange比constrainedSize允许更大的长文本，支持占位符
小心：不支持 layer backing
例子：examples/editableText

## ASImageNode

敬请期待......

ASImageNode性能提示

只需要一行代码就可以方便的查看app没有下载和渲染的过量图形数据，或者低质量的图形数据，使用这个工具pixel scaling debug tool.

## ASNetworkImageNode
ASNetworkImageNode用来展示那些被远程存储的图片，所有你要做的只是设置好URL，需要是一个NSURL实例，并且图片会异步的被夹在，正确的被读取

```objectivec
ASNetworkImageNode *imageNode = [[ASNetworkImageNode alloc] init];
imageNode.URL = [NSURL URLWithString:@"https://someurl.com/image_uri"];
```

### 布局一个网图节点

一个网图节点在还没有真实地大小的时候，是有必要指定一个特定得布局的

方法1：preferredFrameSize

如果你有一个标准大小，你希望这个image节点可以按着布局，你可以使用这个属性

```objectivec
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constraint
{
    imageNode.preferredFrameSize = CGSizeMake(100, 200);
    ...
    return finalLayoutSpec;
}
```

方法2：ASRatioLayoutSpec

这个场景是一个绝好的使用ASRatioLayoutSpec的场景，不去设置静态的大小，你可以指定当图像下载完成时，和图像保持比例，

```objectivec
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constraint
{
    CGFloat ratio = 3.0/1.0;
    ASRatioLayoutSpec *imageRatioSpec = [ASRatioLayoutSpec ratioLayoutSpecWithRatio:ratio child:self.imageNode];
    ...
    return finalLayoutSpec;
}
```

### 外部引用

如果你不打算引入PINRemoteImage和PINCache，你会失去对jpeg的更好的支持，你需要自行引入你自己的cache系统，需要遵从ASImageCacheProtocol

### 渐进式JPEG支持

得益于PINRemoteImage，网图节点可以全面支持，有进度下载的JPEG图片，如果你的服务器提供这个功能，你的图片就可以展示的非常快，先加载低质量图，慢慢展示

逐步加载图片是很重要的，如果服务器被要求使用普通的JPEGS，但是给你提供了多个版本的图片数据，你可以使用ASMultiplexImageNode


### 自动缓存
ASNetworkImageNode使用PINCache 来自动处理网络图片缓存

## ASMultiplexImageNode

如果你不能使用渐进式JPEG，但是你可以处理同一个图的几个不同尺寸的图形数据，你可以使用ASMultiplexImageNode代替ASNetworkImageNode

在下面的例子里，你就是在CellNode里用了一个多图形节点，初始化之后，你通常需要做2个事情，

第一，确保downloadsIntermediateImages设置为YES，这样方便快速下载

第二，分配一个数组里面对应着图片类型key 和 value，这样将帮助节点选择下载哪个URL，尝试加载

```objectivec
- (instancetype)initWithURLs:(NSDictionary *)urls
{
    ...
     _imageURLs = urls;          // something like @{@"thumb": "/smallImageUrl", @"medium": ...}

    _multiplexImageNode = [[ASMultiplexImageNode alloc] initWithCache:nil 
                                                           downloader:[ASBasicImageDownloader sharedImageDownloader]];
    _multiplexImageNode.downloadsIntermediateImages = YES;
    _multiplexImageNode.imageIdentifiers = @[ @"original", @"medium", @"thumb" ];

    _multiplexImageNode.dataSource = self;
    _multiplexImageNode.delegate   = self;
    ...
}
```

然后，如果你已经设置好了一个字典来保存已有的KEY和URL，你就可以简单的返回URL给对应的KEY

```objectivec
#pragma mark Multiplex Image Node Datasource

- (NSURL *)multiplexImageNode:(ASMultiplexImageNode *)imageNode 
        URLForImageIdentifier:(id)imageIdentifier
{
    return _imageURLs[imageIdentifier];
}
```

也有一个delegate可以方便你显示下载进度，展示的时候，你可以根据需求更新你的方法

比如，下面的例子，当你一个新的图片下载完成后，你需要一个回调

```objectivec
#pragma mark Multiplex Image Node Delegate

- (void)multiplexImageNode:(ASMultiplexImageNode *)imageNode 
            didUpdateImage:(UIImage *)image 
            withIdentifier:(id)imageIdentifier 
                 fromImage:(UIImage *)previousImage 
            withIdentifier:(id)previousImageIdentifier;
{    
        // this is optional, in case you want to react to the fact that a new image came in
}
```

## ASMapNode

ASMapNode提供了一个完整的异步准备，自动预加载，高效内存处理的节点
，他的标准模式下，是异步快照的形式，ASTableView 和 ASCollectionView 会自动触发liveMap模式，liveMode模式可以轻松的提供缓存，这是地图交互所必须的

功能：使用MKMapSnapshotOptions所规定的主要形式，允许无缝过度地图快照和3D相机模式，允许滚动时候自动异步加载

不足：MKMapView 不是线程安全的

## ASVideoNode
ASVedioNode是一个新的功能，并且专为方便高效的在滚动试图里嵌入视频

功能：当对象可见，支持自动播放，哪怕是他们在滚动容器内（ASPagerNode or ASTableNode），如果提供了URL缩略图占位符可以异步下载，如果不提供也可以将解码第一帧当做展位图

不足：必须使用AVFoundation 这个库

例子：examples/videosTableview  - examples/videos

## ASScrollNode

敬请期待......

# 排版引擎
## 排版基础

### 盒子模型排版

ASLayout是一个自动的，异步的，纯OC盒子模型排版的布局功能，是一种CSS flex box的简单版，ComponetKit的简化版本，他的目的是让你的布局居右可扩展和复用性

UIView的实例存储位置，大小是通过center和bounds的属性，当约束条件发生变化，CoreAnimation会调用layoutSubviews，告诉view需要更新界面

<ASLayoutable>实例（所有的ASDisplayNodes和子类）不需要大小和位置信息，相反，AsyncDisplayKit会调用layoutSpecThatFits方法通过给一个约束来描述大小和位置信息

### 术语

术语可能有点混乱，所以在这里对所有ASDK自动布局进行简单的说明：

<ASLayoutable>协议定义了测量物体布局的方法，符合<ASLayoutable>协议的对象就有相关的能力。通过ASLayout返回的值，必须有2个前提要确定，1，layoutable对象的大小（不一定是位置），2其所有子节点的大小与位置。通过递推树来让父节点计算布局确定最终的结果。

这个协议是所有布局相关协议的家，包含所有的ASXXLayoutSpec协议，这些协议包含着布局的规则和选项，例如，ASStackLayoutSpec具有限定layoutable如何缩小或放大根据可用空间的作用。这些布局选项都保存在ASLayoutOptions类，一般来说你不需要担心布局选项。如果要创建自定义布局规则，你可以扩展去适应新的布局选项。

所有的ASDisplayNodes和他的子类，以及ASLayoutSpecs都符合这个协议

一个ASLayoutSpec是一个不可变的描述布局的对象，布局规范的创建要通过layoutSpecThatFits：方法，一个布局规范的创建和修改，一旦它传给ASDK，所有的属性都将变成不可变，并且任何设置改变都将导致断言

每个ASLayoutSpec至少要作用在一个孩子上，ASLayoutSpec持有这个孩子，一些约束规则如ASInsetLayoutSpec，只需要一个孩子，其他的规则需要多个孩子。

你不需要了解ASLayout，只需要知道他代表着一个不变的布局树，而且通过遵循<ASLayoutable>协议的对象返回

### UIKit组件布局

- 对于直接添加UIView，你需要手动的在`didLoad:`处理
- 对于添加到ASDK得UIView，你可以在`layoutSpecThatFits:`处理

## 布局容器

AsyncDisplayKit包含有一套布局的组件，下面的LayoutSpecs允许你可以拥有多个孩子

- ASStackLayoutSpec 基于CSS Flexbox的一个简化版本，可以水平或者垂直的排布堆栈组件，并制定如何对其，如何适应空间

- ASStaticLayoutSpec 允许你固定孩子的偏移

下面LayoutSpecs允许你布局单一的孩子

- ASLayoutSpec 可以在没有孩子的时候使用
- ASInsetLayoutSpec 可以处理Inset环绕空余型的孩子
- ASBackgroundLayoutSpec 可以用于拉伸背后一个组件做背景
- ASOverlayLayoutSpec 拼接上面的一个组件
- ASCenterLayoutSpec 中心可用布局
- ASRatioLayoutSpec 固定的比例布局图片GIF视频
- ASRelativeLayoutSpec 9宫格缩放布局

## 布局样例

3个逐渐复杂的样例

### NSSpain Talk例子
[图不翻译了]

```objectivec
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constraint
{
  ASStackLayoutSpec *vStack = [[ASStackLayoutSpec alloc] init];

  [vStack setChildren:@[titleNode, bodyNode];

  ASStackLayoutSpec *hstack = [[ASStackLayoutSpec alloc] init];
  hStack.direction          = ASStackLayoutDirectionHorizontal;
  hStack.spacing            = 5.0;

  [hStack setChildren:@[imageNode, vStack]];

  ASInsetLayoutSpec *insetSpec = [ASInsetLayoutSpec insetLayoutSpecWithInsets:UIEdgeInsetsMake(5,5,5,5) child:hStack];

  return insetSpec;
}
```

### 社交APP布局

[图不翻译了]

```objectivec
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize
{
  // header stack
  _userAvatarImageView.preferredFrameSize = CGSizeMake(USER_IMAGE_HEIGHT, USER_IMAGE_HEIGHT);  // constrain avatar image frame size

  ASLayoutSpec *spacer = [[ASLayoutSpec alloc] init];
  spacer.flexGrow      = YES;

  ASStackLayoutSpec *headerStack = [ASStackLayoutSpec horizontalStackLayoutSpec];
  headerStack.alignItems         = ASStackLayoutAlignItemsCenter;       // center items vertically in horizontal stack
  headerStack.justifyContent     = ASStackLayoutJustifyContentStart;    // justify content to left side of header stack
  headerStack.spacing            = HORIZONTAL_BUFFER;

  [headerStack setChildren:@[_userAvatarImageView, _userNameLabel, spacer, _photoTimeIntervalSincePostLabel]];

  // header inset stack

  UIEdgeInsets insets                = UIEdgeInsetsMake(0, HORIZONTAL_BUFFER, 0, HORIZONTAL_BUFFER);
  ASInsetLayoutSpec *headerWithInset = [ASInsetLayoutSpec insetLayoutSpecWithInsets:insets child:headerStack];
  headerWithInset.flexShrink = YES;

  // vertical stack

  CGFloat cellWidth                  = constrainedSize.max.width;
  _photoImageView.preferredFrameSize = CGSizeMake(cellWidth, cellWidth);  // constrain photo frame size

  ASStackLayoutSpec *verticalStack   = [ASStackLayoutSpec verticalStackLayoutSpec];
  verticalStack.alignItems           = ASStackLayoutAlignItemsStretch;    // stretch headerStack to fill horizontal space

  [verticalStack setChildren:@[headerWithInset, _photoImageView, footerWithInset]];

  return verticalStack;
}
```

完整的ASDK工程可以查阅 example/ASDKgram

### 社交APP布局2

[图不翻译了]

```objectivec
- (ASLayoutSpec *)layoutSpecThatFits:(ASSizeRange)constrainedSize {

  ASLayoutSpec *textSpec  = [self textSpec];
  ASLayoutSpec *imageSpec = [self imageSpecWithSize:constrainedSize];
  ASOverlayLayoutSpec *soldOutOverImage = [ASOverlayLayoutSpec overlayLayoutSpecWithChild:imageSpec 
                                                                                  overlay:[self soldOutLabelSpec]];

  NSArray *stackChildren = @[soldOutOverImage, textSpec];

  ASStackLayoutSpec *mainStack = [ASStackLayoutSpec stackLayoutSpecWithDirection:ASStackLayoutDirectionVertical 
                                                                         spacing:0.0
                                                                  justifyContent:ASStackLayoutJustifyContentStart
                                                                      alignItems:ASStackLayoutAlignItemsStretch          
                                                                        children:stackChildren];

  ASOverlayLayoutSpec *soldOutOverlay = [ASOverlayLayoutSpec overlayLayoutSpecWithChild:mainStack 
                                                                                overlay:self.soldOutOverlay];

  return soldOutOverlay;
}

- (ASLayoutSpec *)textSpec {
  CGFloat kInsetHorizontal        = 16.0;
  CGFloat kInsetTop               = 6.0;
  CGFloat kInsetBottom            = 0.0;
  UIEdgeInsets textInsets         = UIEdgeInsetsMake(kInsetTop, kInsetHorizontal, kInsetBottom, kInsetHorizontal);

  ASLayoutSpec *verticalSpacer    = [[ASLayoutSpec alloc] init];
  verticalSpacer.flexGrow         = YES;

  ASLayoutSpec *horizontalSpacer1 = [[ASLayoutSpec alloc] init];
  horizontalSpacer1.flexGrow      = YES;

  ASLayoutSpec *horizontalSpacer2 = [[ASLayoutSpec alloc] init];
  horizontalSpacer2.flexGrow      = YES;

  NSArray *info1Children = @[self.firstInfoLabel, self.distanceLabel, horizontalSpacer1, self.originalPriceLabel];
  NSArray *info2Children = @[self.secondInfoLabel, horizontalSpacer2, self.finalPriceLabel];
  if ([ItemNode isRTL]) {
    info1Children = [[info1Children reverseObjectEnumerator] allObjects];
    info2Children = [[info2Children reverseObjectEnumerator] allObjects];
  }

  ASStackLayoutSpec *info1Stack = [ASStackLayoutSpec stackLayoutSpecWithDirection:ASStackLayoutDirectionHorizontal 
                                                                          spacing:1.0
                                                                   justifyContent:ASStackLayoutJustifyContentStart 
                                                                       alignItems:ASStackLayoutAlignItemsBaselineLast children:info1Children];

  ASStackLayoutSpec *info2Stack = [ASStackLayoutSpec stackLayoutSpecWithDirection:ASStackLayoutDirectionHorizontal 
                                                                          spacing:0.0
                                                                   justifyContent:ASStackLayoutJustifyContentCenter 
                                                                       alignItems:ASStackLayoutAlignItemsBaselineLast children:info2Children];

  ASStackLayoutSpec *textStack = [ASStackLayoutSpec stackLayoutSpecWithDirection:ASStackLayoutDirectionVertical 
                                                                         spacing:0.0
                                                                  justifyContent:ASStackLayoutJustifyContentEnd
                                                                      alignItems:ASStackLayoutAlignItemsStretch
                                                                        children:@[self.titleLabel, verticalSpacer, info1Stack, info2Stack]];

  ASInsetLayoutSpec *textWrapper = [ASInsetLayoutSpec insetLayoutSpecWithInsets:textInsets 
                                                                          child:textStack];
  textWrapper.flexGrow = YES;

  return textWrapper;
}

- (ASLayoutSpec *)imageSpecWithSize:(ASSizeRange)constrainedSize {
  CGFloat imageRatio = [self imageRatioFromSize:constrainedSize.max];

  ASRatioLayoutSpec *imagePlace = [ASRatioLayoutSpec ratioLayoutSpecWithRatio:imageRatio child:self.dealImageView];

  self.badge.layoutPosition = CGPointMake(0, constrainedSize.max.height - kFixedLabelsAreaHeight - kBadgeHeight);
  self.badge.sizeRange = ASRelativeSizeRangeMake(ASRelativeSizeMake(ASRelativeDimensionMakeWithPercent(0), ASRelativeDimensionMakeWithPoints(kBadgeHeight)), ASRelativeSizeMake(ASRelativeDimensionMakeWithPercent(1), ASRelativeDimensionMakeWithPoints(kBadgeHeight)));
  ASStaticLayoutSpec *badgePosition = [ASStaticLayoutSpec staticLayoutSpecWithChildren:@[self.badge]];

  ASOverlayLayoutSpec *badgeOverImage = [ASOverlayLayoutSpec overlayLayoutSpecWithChild:imagePlace overlay:badgePosition];
  badgeOverImage.flexGrow = YES;

  return badgeOverImage;
}

- (ASLayoutSpec *)soldOutLabelSpec {
  ASCenterLayoutSpec *centerSoldOutLabel = [ASCenterLayoutSpec centerLayoutSpecWithCenteringOptions:ASCenterLayoutSpecCenteringXY 
  sizingOptions:ASCenterLayoutSpecSizingOptionMinimumXY child:self.soldOutLabelFlat];
  ASStaticLayoutSpec *soldOutBG = [ASStaticLayoutSpec staticLayoutSpecWithChildren:@[self.soldOutLabelBackground]];
  ASCenterLayoutSpec *centerSoldOut = [ASCenterLayoutSpec centerLayoutSpecWithCenteringOptions:ASCenterLayoutSpecCenteringXY   sizingOptions:ASCenterLayoutSpecSizingOptionDefault child:soldOutBG];
  ASBackgroundLayoutSpec *soldOutLabelOverBackground = [ASBackgroundLayoutSpec backgroundLayoutSpecWithChild:centerSoldOutLabel background:centerSoldOut];
  return soldOutLabelOverBackground;
}
```

完整的ASDK工程可以查阅 example/CatDealsCollectionView

## 布局调试

使用ASC II Art 调试

ASLayoutSpecPlayground App

## 布局选项
当使用ASDK的时候，你有3种布局选择，注意：UIKit的autolayout不支持

### 手动计算布局

最原始的布局方式，类似于UIKit的布局方法，ASViewControllers使用这种布局方法

[ASDisplayNode calculateSizeThatFits:] vs. [UIView sizeThatFits:]

[ASDisplayNode layout] vs. [UIView layoutSubviews]

优势：（针对UIKit）

- 消除了所有主线程布局的开销
- 结果缓存

缺点：（针对UIKit）

- 代码之间会有重复代码
- 逻辑不可重用

# 优化
## 层处理 Layer-Backing

有些时候，大幅度使用Layer而不是使用views，可以提高你的app的性能，但是手动的把基于view开发的界面代码改为基于layer的界面代码，非常的费劲，如果有时候因为要开启触摸或者view特定的功能的时候，你可能要功能回退

当你使用ASDK的node的时候，如果你打算把所有的view转换成layer，只需要一行代码

`rootNode.layerBacked = YES;`

如果你想回退，也只需要删除这一行，我们建议不需要触摸处理的所有视图都开启

## 同步并发

敬请期待......

## 子树光栅化

预压缩，扁平化整个视图层级到一个图层，也可以提高性能，node也可以帮你做这件事

`rootNode.shouldRasterizeDescendants = YES;`

你的整个node层级都会渲染在一个layer下

## 绘制优先级

敬请期待......

# 开发工具

## 点击区域扩展

ASDisplayNode有一个hitTestSlop属性，是UIEdgeInsets，当这个值非零的时候，可以增加点击区域，更加方便进行点击

ASDisplayNode是所有节点的基类，所以这个属性可以在任何node上使用

注意：
这会影响-hitTest和-pointInside的默认实现，如果子类需要调用，请忽略

节点的触摸事件受到其父的边界+父HitTestSlop限制，如果想扩展父节点下的一个孩子节点的边界，请直接扩展父节点

## 批量获取API
ASDK的批量获取API可以很方便的让开发者获取大量新数据，如果用户滚动一个列表或者宫格的view，会自动的在特定范围内批量抓取，时机是由ASDK触发的

作为开发者，可以定义批量抓取的时机，ASTableView和ASCollectionView有个leadingScreensForBatching属性，用来处理这个，默认是2.0

为了实现批量抓取，必须实现2个delegate

`- (BOOL)shouldBatchFetchForTableView:(ASTableView *)tableView`

或者

`- (BOOL)shouldBatchFetchForCollectionView:(ASCollectionView *)collectionView`

在这两个方法你来决定当用户滚动到一定范围的时候，批量获取是否启动。一般是基于数据是否已经抓取完毕，或者本地操作来决定的

如果`- (BOOL)shouldBatchFetchForCollectionView:(ASCollectionView *)collectionView,`返回NO，就不会产生新的批量抓取处理，如果返回YES，批量抓取就会开始，会调用下面2个方法

`- (void)tableView:(ASTableView *)tableView willBeginBatchFetchWithContext:(ASBatchContext *)context;`

或者

`- (void)collectionView:(ASCollectionView *)collectionView willBeginBatchFetchWithContext:(ASBatchContext *)context;`

首先你要小心，这两个方法是在后台线程被调用的，如果你必须在主线程上操作，你就得把它分派到主线程去完成这些操作

当你完成数据读取后，要让ASDK知道你已经完成了，必须调用completeBatchFetching:，并且传YES,这就确保整批提取机制保持同步，下一次提取循环可以正常工作，只有传YES上下文才知道在必要的时候准备下一次更新，如果传NO，什么都不会发生

批量获取demo
```objectivec
- (BOOL)shouldBatchFetchForTableView:(ASTableView *)tableView 
{
  // Decide if the batch fetching mechanism should kick in
  if (_stillDataToFetch) {
    return YES;
  }
  return NO;
}

- (void)tableView:(ASTableView *)tableView willBeginBatchFetchWithContext:(ASBatchContext *)context 
{
  // Fetch data most of the time asynchronoulsy from an API or local database
  NSArray *data = ...;

  // Insert data into table or collection view
  [self insertNewRowsInTableView:newPhotos];

  // Decide if it's still necessary to trigger more batch fetches in the future
  _stillDataToFetch = ...;

  // Properly finish the batch fetch
  [context completeBatchFetching:YES]
}
```

查看更多demo可以看

- ASDKgram
- Kittens
- CatDealsCollectionView
- ASCollectionView


## 图片修改块

敬请期待......

## 占位符渐隐

敬请期待......

## 点击区域可视
### 可视化的点击区域

这是一个调试功能，把任何的ASControlNodes加上半透明高亮，点击，手势识别，这个范围定义为ASControlNodes的frame+hitTestSlop的范围

在下面的截图中你可以看到

- 头像图片区域+用户名区域可点击，
- hitTestSlop扩大的点击范围，可以显示出来，更容易点击
- hitTestSlop缩小的点击范围，也可以观察出来

[图不翻译了]

### 限制
在收到父节点clipsToBounds的剪裁

### 用法
在你的Appdelegate.m中
添加[ASControlNode setEnableHitTestDebug:YES] 到你的didFinishLaunchingWithOptions: 方法的最上方，
确保在任何ASControllNode初始化前调用这个方法，包括ASButtonNodes, ASImageNodes, and ASTextNodes.

## 图片缩放

可视化的ASImageNode.image像素缩放
如果像素图像不符合像素大小，这个调试工具会增加了一个红色的文本出现在ASImageNode右下角，

```objectivec
imageSizeInPixels = image.size * image.scale
boundsSizeInPixels = bounds.size * contentsScale
scaleFactor = imageSizeInPixels / boundsSizeInPixels

if (scaleFactor != 1.0) {
      NSString *scaleString = [NSString stringWithFormat:@"%.2fx", scaleFactor];
      _debugLabelNode.hidden = NO;
}
```
此调试工具在下面的情况非常有用

- 下载和展现的图像数据过多
- 放大低质量的图片

在下面的截图中，你可以看到，低质量图片被放大因此右下角有文字，你需要优化你的功能，控制最终的尺寸和最佳的图像

[图不翻译了]

### 使用

在appdelegate.m文件中

导入AsyncDisplayKit+Debug.h

添加[ASImageNode setShouldShowImageScalingOverlay:YES] 到didFinishLaunchingWithOptions: 方法的最上方


# 测试中的功能
## 隐藏节点层次管理

先不翻译了吧。。未稳定的功能

## 排版动画API

先不翻译了把。。未稳定的功能