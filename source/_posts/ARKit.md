---
title: ARKit教程
date: 2018-09-08 9:56:00
categories:
- 编程
- IOS
tags:
- 编程
- IOS
copyright:
---

<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

# ARKit教程

## 背景介绍

苹果公司在WWDC2017 上发布了ARKit,开发者可以使用这套框架在iPhone和iPad上创建属于自己的AR体验。

增强现实技术（Augmented Reality，简称 AR），是一种实时地计算摄影机影像的位置及角度并加上相应图像、视频、3D模型的技术，这种技术的目标是在屏幕上把虚拟世界套在现实世界并进行互动。

ARKit框架提供了两种AR技术，一种是基于3D场景(SceneKit)实现的增强现实，一种是基于2D场景(SpriktKit)实现的增强现实。

需要注意的是，ARKit虽说是iOS11提供的框架，但是并非升级到iOS11都能使用。必须要A9以及以上的处理器才能使用。以下是ARKit 开发环境：

1.Xcode版本：Xcode9及以上

2.iOS系统:iOS11及以上

3.iOS设备：处理器A9及以上（6S机型及以上）

4.MacOS系统：10.12.4及以上（安装Xcode9对Mac系统版本有要求）

> 根据ARKit的描述，兼容苹果AR功能的iPhone必须搭载iOS 11，同时对手机、平板的整体性能也有更高要求。ARKit支持所有安装了ios 11的iphone与ipad。

判断iPhone 硬件设备型号:

- iPhone8,1 iPhone6S

- iPhone8,2 iPhone6S Plus

- iPhone8,4 iPhoneSE

- iPhone9,1 /iPhone9,3 iPhone 7

- iPhone9,2/iPhone9,4 iPhone 7 Plus 

- iPhone10,1/iPhone10,4          iPhone 8

- iPhone10,2/iPhone10,5          iPhone 8 Plus

- iPhone10,3/iPhone10,6   iPhone X

## 核心功能

（1）追踪

追踪是 ARKit 的核心功能，可以获得设备在物理世界中的方向和位置，此外还可以追踪物体（例如人脸）。

（2）场景理解

场景理解通过学习环境的一些属性来优化追踪，检测水平面（例如地面和桌面）和垂直面以便在场景中放置物体。场景理解还会学习环境中的光照情况，以便在虚拟场景中精确模拟真实的光照，以防物体看起来过亮或过暗。

（3）渲染

渲染就是指用户在设备上实际看到、并且在 AR 场景中交互的内容。ARKit 可以很方便的与你喜欢的渲染引擎进行集成，还为 SceneKit 和 SpriteKit 提供了内置视图。此外 Xcode 还提供了可以快速上手的 Metal 模板。

## ARKit工作原理和流程

首先需要说明的是，ARKit并不是一个能够独立运行的框架，其依赖SceneKit框架。简单点说，ARKit框架主要提供相机相关的工作，主要负责捕捉和分析现实世界。而展示虚拟物体部分，则是依赖SceneKit提供的能力。如果没有SceneKit，那ARKit只是一个相机而已。（SceneKit是苹果在iOS8中集成的一套3D引擎框架）

下图是ARKit中主要类的关系图：

![](http://p55se4hrx.bkt.clouddn.com/images/programmings/ARKit/ARKit_01.jpg)

上图中，ARSCNView可以认为是一个容器，代表的就是看到的现实世界。其作用有两个：

1、如上图右边部分：管理ARSession，在这里简单认为ARSession是管理ARKit世界的上下文，其管理者相机的属性设置，也负责向ARSCNView输出捕捉到的显示世界。

2、如上图左边部分：显示我们添加进去的AR物体，这里可以是一个3D物体，也可以是一个2D物体。这里从继承关系来看，ARSCNView、SCNScene、SCNNode等类是从SceneKit中继承过来的。

从上图可以简单的看出ARKit的工作原理，ARKit框架提供相机能力，在手机上捕捉并构建现实世界。SceneKit提供模型能力，在手机的“现实世界”中添加虚拟物体。

## SceneKit 简介

### 坐标系

UIKit 使用一个包含有 x 和 y 信息的 CGPoint 来表示一个点的位置，但是在 3D 系统中，需要一个 z 参数来描述物体在空间中的深度，SceneKit 的坐标系可以参考下图：

![](http://p55se4hrx.bkt.clouddn.com/images/programmings/ARKit/ARKit_05_SceneKit.png)

这个三维坐标系中，表示一个点的位置需要使用(x,y,z)坐标表示。红色方块位于 x 轴，绿色方块位于 y 轴，蓝色方块位于 z 轴，灰色方块位于原点。在 SceneKit 中我们可以这样创建一个三维坐标：

```swift
let position = SCNVector3(x: 0, y: 5, z: 10)
```

### SceneKit 中的场景和节点

SceneKit 中的场景(SCNScene)想象为一个虚拟的 3D 空间，然后可以将一个个的节点(SCNNode)添加到场景中。SCNScene 中有唯一一个根节点(坐标是(x:0, y:0, z:0))，除了根节点外，所有添加到 SCNScene 中的节点都需要一个父节点。

SCNScene 中的节点加入时可以指定一个三维坐标(默认为(x:0, y:0, z:0))，这个坐标是相对于其父节点的位置。这里说明两个概念：

- 本地坐标系：以场景中的某节点(非根节点)为原点建立的三维坐标系
- 世界坐标系：以根节点为原点创建的三维坐标系称为世界坐标系。

### SceneKit 中的摄像机

有了 SCNScene 和 SCNNode 后，我们还需要一个摄像机(SCNCamera)来决定我们可以看到场景中的哪一块区域(就好比现实世界中有了各种物体，但还需要人的眼睛才能看到物体)。摄像机在 SCNScene 的工作模式如下图：

![](http://p55se4hrx.bkt.clouddn.com/images/programmings/ARKit/ARKit_06_SceneKit.png)

上图中包含以下几点信息：

- SceneKit 中 SCNCamera 拍摄的方向始终为 z 轴负方向。
- 视野(Field of View)是摄像机的可视区域的极限角度。角度越小，视野越窄，反之，角度越大，视野越宽。
- 视锥体(Viewing Frustum)决定着摄像头可视区域的深度(z 轴表示深度)。任何不在这个区域内的物体将被剪裁掉(离摄像头太近或者太远)，不会显示在最终的画面中。

在 SceneKit 中我们可以使用如下方式创建一个摄像机：

```swift
let scene = SCNScene()
let cameraNode = SCNNode()
let camera = SCNCamera()
cameraNode.camera = camera
cameraNode.position = SCNVector3(x: 0, y: 0, z: 0)
scene.rootNode.addChildNode(cameraNode)
```

### SCNView

最后，我们需要一个 View 来将 SCNScene 中的内容渲染到显示屏幕上，这个工作由 SCNView 完成。这一步其实很简单，只需要创建一个 SCNView 实例，然后将 SCNView 的 scene 属性设置为刚刚创建的 SCNScene，然后将 SCNView 添加到 UIKit 的 view 或 window 上即可。示例代码如下：

```swift
let scnView = SCNView()
scnView.scene = scene
vc.view.addSubview(scnView)
scnView.frame = vc.view.bounds
```

## ARKit API介绍

ARKit框架的API其实并不多。下图就是整个ARKit框架提供的类。下面就这几个主要的类（介绍主要的属性）做一个简单的说明。

![](http://p55se4hrx.bkt.clouddn.com/images/programmings/ARKit/ARKit_02.jpg)

### （1）ARAnchor

ARAnchor表示一个物体在3D空间的位置和方向（ARAnchor通常称为物体的3D锚点，有点像UIKit框架中CALayer的Anchor）.

注：ARFrame表示的也是物体的位置和方向，但是ARFrame通常表示的是AR相机的位置和方向以及追踪相机的时间，还可以捕捉相机的帧图片,也就是说ARFrame用于捕捉相机的移动，其他虚拟物体用ARAnchor。

### （2）ARCamera

ARCamera是一个相机，它是连接虚拟场景与现实场景之间的枢纽。在ARKit中，它是捕捉现实图像的相机，在SceneKit中它又是3D虚拟世界中的相机。ARCamera 是 ARFrame 中的一个属性，之因为单独拿出来说，是因为这里有必要介绍下相机的一些特性。ARCamera 中与现实世界场景有关的信息有两个：

- var imageResolution: CGSize
该属性表示了相机捕捉到的图像的长度和宽度(以像素为单位)，可以理解成捕捉到的图像的分辨率。

- var intrinsics: matrix_float3x3
intrinsics 是一个 3x3 矩阵，这个矩阵将我们现实世界中三维坐标系的点映射到相机捕捉的图像中。有兴趣可看下面的详述。

Intrinsic Matrix：

Intrinsic Matrix 是相机的一个固有属性，也就是说每个相机都会有 Intrinsic Matrix，因为所有的相机都需要将现实世界中三维空间的点映射到捕捉的图像中二维空间的点。那么这个矩阵是如何工作的呢？我们先来看一个图片：

![](http://p55se4hrx.bkt.clouddn.com/images/programmings/ARKit/ARKit_04_ARCamera.png)

上图包含如下基本信息：

- 一个三维坐标系(红色 x 轴，绿色 y 轴，蓝色 z 轴)。
- 空间中的一个点(蓝色的点 N，坐标为(x', y', z'))。
- 相机的成像平面(紫色的平行四边形)
- 成像平面与 z 轴的交点(点 M)
- 成像平面的原点(黄色的点 O)，也就是捕捉的二维图像的二维坐标系的原点。

现在我们需要将三维空间的点(x', y', z')映射到成像平面中的一个点(N')。

- 一般我们无需去创建一个相机，因为当我们初始化一个AR视图时，会为我们默认创建一个相机，而且这个相机就是摄像头的位置，同时也是3D世界中的原点所在（x=0,y=0,z=0）。
- ARCamera的API一般我们无需关心，因为ARKit会默认帮助我们配置好.

```OC
@interface ARCamera : NSObject <NSCopying>

/**
 4x4矩阵表示相机位置，同ARAnchor
 */
@property (nonatomic, readonly) matrix_float4x4 transform;

/**
相机方向（旋转）的矢量欧拉角
分别是x/y/z
 */
@property (nonatomic, readonly) vector_float3 eulerAngles;

/**
 相机追踪状态（在下方会有枚举值介绍）
 */
@property (nonatomic, readonly) ARTrackingState trackingState NS_REFINED_FOR_SWIFT;

/**
追踪运动类型
 */
@property (nonatomic, readonly) ARTrackingStateReason trackingStateReason NS_REFINED_FOR_SWIFT;

/**
相机内置函数
3x3矩阵
 fx 0   px
 0  fy  py
 0  0   1
 fx和fy是像素焦距
 px和py是主像素点坐标
 原点位于左上像素的中心
 */
@property (nonatomic, readonly) matrix_float3x3 intrinsics;

/**
摄像头分辨率
 */
@property (nonatomic, readonly) CGSize imageResolution;

/**
投影矩阵
*/
@property (nonatomic, readonly) matrix_float4x4 projectionMatrix;

/**
返回一个二维坐标点
*/
- (CGPoint)projectPoint:(vector_float3)point orientation:(UIInterfaceOrientation)orientation viewportSize:(CGSize)viewportSize;

/**
创建相机投影矩阵
 */
- (matrix_float4x4)projectionMatrixWithViewportSize:(CGSize)viewportSize orientation:(UIInterfaceOrientation)orientation zNear:(CGFloat)zNear zFar:(CGFloat)zFar;

@end

//相机追踪状态枚举
typedef NS_ENUM(NSInteger, ARTrackingState) {

    /** 不被允许 */
    ARTrackingStateNotAvailable,

    /** 受限状态 */
    ARTrackingStateLimited,

    /** 正常. */
    ARTrackingStateNormal,
} NS_REFINED_FOR_SWIFT;

/**
 描述相机追踪状态有限的原因
 */
typedef NS_ENUM(NSInteger, ARTrackingStateReason) {
    /** 无. */
    ARTrackingStateReasonNone,
    
    /** 正在初始化，导致跟踪受限 */
    ARTrackingStateReasonInitializing,

    /** 运动过度. */
    ARTrackingStateReasonExcessiveMotion,

    /** 缺少可见的特征. */
    ARTrackingStateReasonInsufficientFeatures,
} NS_REFINED_FOR_SWIFT;
```

### (3)ARError

ARError是一个描述ARKit错误的类，这个错误来源于几个方面，例如设备不支持，或者当相机常驻后台时ARSession会断开等问题.

```OC
//作用域，一般会表示是哪一个类出现问题
NSString *const ARErrorDomain;

//错误码描述  100：不支持会话追踪配置，主线由于A9芯片以下的机型会报错   101：失活状态 102：传感器故障  200：追踪失败
typedef NS_ERROR_ENUM(ARErrorDomain, ARErrorCode) {
    /** Unsupported session configuration. */
    ARErrorCodeUnsupportedConfiguration   = 100,

    /** A sensor required to run the session is not available. */
    ARErrorCodeSensorUnavailable          = 101,

    /** A sensor failed to provide the required input. */
    ARErrorCodeSensorFailed               = 102,

    /** World tracking has encountered a fatal error. */
    ARErrorCodeWorldTrackingFailed        = 200,
};
```

### (4) ARHitTestResult

ARHitTestResult：点击回调结果，这个类主要用于虚拟增强现实技术（AR技术）中现实世界与3D场景中虚拟物体的交互。 比如我们在相机中移动。拖拽3D虚拟物体，都可以通过这个类来获取ARKit所捕捉的结果.

```OC
//捕捉类型枚举
typedef NS_OPTIONS(NSUInteger, ARHitTestResultType) {
    /** 点. */
    ARHitTestResultTypeFeaturePoint              = (1 << 0),

    /** 水平面 y为0. */
    ARHitTestResultTypeEstimatedHorizontalPlane  = (1 << 1),

    /** 已结存在的平面. */
    ARHitTestResultTypeExistingPlane             = (1 << 3),

    /** 已结存在的锚点和平面. */
    ARHitTestResultTypeExistingPlaneUsingExtent  = (1 << 4),
} NS_SWIFT_NAME(ARHitTestResult.ResultType);

/**
捕捉类型
 */
@property (nonatomic, readonly) ARHitTestResultType type;

/**
 3D虚拟物体与相机的距离（单位：米）
 */
@property (nonatomic, readonly) CGFloat distance;

/**
本地坐标矩阵（世界坐标指的是相机为场景原点的坐标，而每一个3D物体自身有一个场景，本地坐标就是相对于这个场景的坐标）类似于frame和bounds的区别
 */
@property (nonatomic, readonly) matrix_float4x4 localTransform;

/**
世界坐标矩阵
 */
@property (nonatomic, readonly) matrix_float4x4 worldTransform;

/**
 锚点（3D虚拟物体，在虚拟世界有一个位置，这个位置参数是SceneKit中的SCNVector3：三维矢量），而锚点anchor是这个物体在AR现实场景中的位置，是一个4x4的矩阵
 */
@property (nonatomic, strong, nullable, readonly) ARAnchor *anchor;
```

### (5) ARLightEstimate

ARLightEstimate是一个灯光效果，它可以让你的AR场景看起来更加的好.

```OC
@interface ARLightEstimate : NSObject <NSCopying>

/**
灯光强度  范围0-2000 默认1000
 */
@property (nonatomic, readonly) CGFloat ambientIntensity;

@end
```

### (6) ARFrame

ARFrame主要是追踪相机当前的状态，这个状态不仅仅只是位置，还有图像帧及时间等参数:

```OC
@interface ARFrame : NSObject <NSCopying>

/**
时间戳.
 */
@property (nonatomic, readonly) NSTimeInterval timestamp;

/**
 缓冲区图像帧
 */
@property (nonatomic, readonly) CVPixelBufferRef capturedImage;

/**
相机（表示这个ARFrame是哪一个相机的，iPhone7plus有两个摄像机）
 */
@property (nonatomic, copy, readonly) ARCamera *camera;

/**
 返回当前相机捕捉到的锚点数据（当一个3D虚拟模型加入到ARKit中时，锚点值得就是这个模型在AR中的位置）
 */
@property (nonatomic, copy, readonly) NSArray<ARAnchor *> *anchors;

/**
灯光，详情可见本章节ARLightEstimate类介绍（指的是灯光强度 一般是0-2000，系统默认1000）
 */
@property (nonatomic, copy, nullable, readonly) ARLightEstimate *lightEstimate;

/**
特征点（应该是捕捉平地或者人脸的，比较苹果有自带的人脸识别功能）
 */
@property (nonatomic, nullable, readonly) ARPointCloud *rawFeaturePoints;

/**
根据2D坐标点搜索3D模型，这个方法通常用于，当我们在手机屏幕点击某一个点的时候，可以捕捉到这一个点所在的3D模型的位置，至于为什么是一个数组非常好理解。手机屏幕一个是长方形，这是一个二维空间。而相机捕捉到的是一个由这个二维空间射出去的长方体，我们点击屏幕一个点可以理解为在这个长方体的边缘射出一条线，这一条线上可能会有多个3D物体模型
point：2D坐标点（手机屏幕某一点）
ARHitTestResultType：捕捉类型  点还是面
(NSArray<ARHitTestResult *> *)：追踪结果数组  详情见本章节ARHitTestResult类介绍

 */
- (NSArray<ARHitTestResult *> *)hitTest:(CGPoint)point types:(ARHitTestResultType)types;

/**
相机窗口的的坐标变换（可用于相机横竖屏的旋转适配）
 */
- (CGAffineTransform)displayTransformWithViewportSize:(CGSize)viewportSize orientation:(UIInterfaceOrientation)orientation;

@end
```

### (7) ARPlaneAnchor

ARPlaneAnchor是ARAnchor的子类，笔者称之为平地锚点。ARKit能够自动识别平地，并且会默认添加一个锚点到场景中，当然要想看到真实世界中的平地效果，需要我们自己使用SCNNode来渲染这个锚点,锚点只是一个位置.

```OC
/**
平地类型，目前只有一个，就是水平面
 */
@property (nonatomic, readonly) ARPlaneAnchorAlignment alignment;

/**
3轴矢量结构体，表示平地的中心点  x/y/z
 */
@property (nonatomic, readonly) vector_float3 center;

/**
3轴矢量结构体，表示平地的大小（宽度和高度）  x/y/z
 */
@property (nonatomic, readonly) vector_float3 extent;

@end
```

### (8) ARPointCloud

ARPointCloud：点状渲染云，主要用于渲染场景.

```OC
@interface ARPointCloud : NSObject <NSCopying>

/**
 点的数量
 */
@property (nonatomic, readonly) NSUInteger count;

/**
每一个点的位置的集合（结构体带*表示的是结构体数组）
 */
@property (nonatomic, readonly) const vector_float3 *points;

@end
```

### (9) ARSCNView

ARKit支持3D的AR场景和2D的AR场景，ARSCNView是3D的AR场景视图:

- 该类非常重要，且API较多
- 该类是整个ARKit框架中唯一两个有代理的类其中之一

```OC
@interface ARSCNView : SCNView

/**
代理
 */
@property (nonatomic, weak, nullable) id<ARSCNViewDelegate> delegate;

/**
AR会话
 */
@property (nonatomic, strong) ARSession *session;

/**
场景
 */
@property(nonatomic, strong) SCNScene *scene;

/**
是否自动适应灯光
 */
@property(nonatomic) BOOL automaticallyUpdatesLighting;

/**
返回对应节点的锚点，节点是一个3D虚拟物体，它的坐标是虚拟场景中的坐标，而锚点ARAnchor是ARKit中现实世界的坐标。
 */
- (nullable ARAnchor *)anchorForNode:(SCNNode *)node;

/**
返回对应锚点的物体
 */
- (nullable SCNNode *)nodeForAnchor:(ARAnchor *)anchor;

/**
根据2D坐标点搜索3D模型，这个方法通常用于，当我们在手机屏幕点击某一个点的时候，可以捕捉到这一个点所在的3D模型的位置，至于为什么是一个数组非常好理解。手机屏幕一个是长方形，这是一个二维空间。而相机捕捉到的是一个由这个二维空间射出去的长方体，我们点击屏幕一个点可以理解为在这个长方体的边缘射出一条线，这一条线上可能会有多个3D物体模型
point：2D坐标点（手机屏幕某一点）
ARHitTestResultType：捕捉类型  点还是面
(NSArray<ARHitTestResult *> *)：追踪结果数组  详情见本章节ARHitTestResult类介绍
数组的结果排序是由近到远
 */
- (NSArray<ARHitTestResult *> *)hitTest:(CGPoint)point types:(ARHitTestResultType)types;

@end

//代理
#pragma mark - ARSCNViewDelegate


//代理的内部实现了SCNSceneRendererDelegate：scenekit代理 和ARSessionObserver：ARSession监听（KVO机制）
@protocol ARSCNViewDelegate <SCNSceneRendererDelegate, ARSessionObserver>
@optional

/**
自定义节点的锚点
 */
- (nullable SCNNode *)renderer:(id <SCNSceneRenderer>)renderer nodeForAnchor:(ARAnchor *)anchor;

/**
当添加节点是会调用，我们可以通过这个代理方法得知我们添加一个虚拟物体到AR场景下的锚点（AR现实世界中的坐标）
 */
- (void)renderer:(id <SCNSceneRenderer>)renderer didAddNode:(SCNNode *)node forAnchor:(ARAnchor *)anchor;

/**
将要刷新节点
 */
- (void)renderer:(id <SCNSceneRenderer>)renderer willUpdateNode:(SCNNode *)node forAnchor:(ARAnchor *)anchor;

/**
 已经刷新节点
 */
- (void)renderer:(id <SCNSceneRenderer>)renderer didUpdateNode:(SCNNode *)node forAnchor:(ARAnchor *)anchor;

/**
 移除节点
 */
- (void)renderer:(id <SCNSceneRenderer>)renderer didRemoveNode:(SCNNode *)node forAnchor:(ARAnchor *)anchor;

@end
```

### (10) ARSession

ARSession是一个连接底层与AR视图之间的桥梁，其实ARSCNView内部所有的代理方法都是由ARSession来提供的.

ARSession与ARScnView之间的关系看起来是这样的:

![](http://p55se4hrx.bkt.clouddn.com/images/programmings/ARKit/ARKit_03_ARSeesion.png)

ARSession获取相机位置数据主要有两种方式:

第一种：push。 实时不断的获取相机位置，由ARSession主动告知用户。通过实现ARSession的代理
-(void)session:(ARSession *)session didUpdateFrame:(ARFrame *)frame来获取.

第二种：pull。 用户想要时，主动去获取。ARSession的属性currentFrame来获取.

```OC
@interface ARSession : NSObject

/**
 代理
 */
@property (nonatomic, weak) id <ARSessionDelegate> delegate;

/**
指定代理执行的线程（主线程不会有延迟，子线程会有延迟），不指定的话默认主线程
 */
@property (nonatomic, strong, nullable) dispatch_queue_t delegateQueue;

/**
相机当前的位置（是由会话追踪配置计算出来的）
 */
@property (nonatomic, copy, nullable, readonly) ARFrame *currentFrame;

/**
 会话追踪配置
 */
@property (nonatomic, copy, nullable, readonly) ARConfiguration *configuration;

/**
运行会话（这行代码就是开启AR的关键所在）
 */
- (void)runWithConfiguration:(ARConfiguration *)configuration NS_SWIFT_UNAVAILABLE("Use run(_:options:)");

/**
运行会话，只是多了一个参数ARSessionRunOptions：作用就是会话断开重连时的行为。是一个枚举值如下：  
ARSessionRunOptionResetTracking：表示重置追踪
ARSessionRunOptionRemoveExistingAnchors：移除现有锚点
 */
- (void)runWithConfiguration:(ARSessionConfiguration *)configuration options:(ARSessionRunOptions)options NS_SWIFT_NAME(run(_:options:));

/**
暂停会话
 */
- (void)pause;

/**
添加锚点
 */
- (void)addAnchor:(ARAnchor *)anchor NS_SWIFT_NAME(add(anchor:));

/**
移除锚点
 */
- (void)removeAnchor:(ARAnchor *)anchor NS_SWIFT_NAME(remove(anchor:));

@end

//session代理分类两部分，一个是观察者（KVO） 一个是委托者（代理）
#pragma mark - ARSessionObserver


//session KVO观察者
@protocol ARSessionObserver <NSObject>

@optional

/**
 session失败
 */
- (void)session:(ARSession *)session didFailWithError:(NSError *)error;

/**
相机改变追踪状态
 */
- (void)session:(ARSession *)session cameraDidChangeTrackingState:(ARCamera *)camera;

/**
 session意外断开（如果开启ARSession之后，APP退到后台就有可能导致会话断开）
 */
- (void)sessionWasInterrupted:(ARSession *)session;

/**
session会话断开恢复（短时间退到后台再进入APP会自动恢复）
 */
- (void)sessionInterruptionEnded:(ARSession *)session;

@end

#pragma mark - ARSessionDelegate



@protocol ARSessionDelegate <ARSessionObserver>

@optional

/**
 相机当前状态（ARFrame：空间位置，图像帧等）更新
 */
- (void)session:(ARSession *)session didUpdateFrame:(ARFrame *)frame;

/**
添加锚点
 */
- (void)session:(ARSession *)session didAddAnchors:(NSArray<ARAnchor*>*)anchors;

/**
刷新锚点
 */
- (void)session:(ARSession *)session didUpdateAnchors:(NSArray<ARAnchor*>*)anchors;

/**
移除锚点
 */
- (void)session:(ARSession *)session didRemoveAnchors:(NSArray<ARAnchor*>*)anchors;

@end
```

### (11) ARConfiguration

注意：该类还有一个子类：ARWorldTrackingConfiguration，它们在同一个API文件中.

```OC
//会话追踪配置类
@interface ARConfiguration : NSObject <NSCopying>

/**
当前设备是否支持，一般A9芯片以下设备不支持
 */
@property(class, nonatomic, readonly) BOOL isSupported;

/**
会话的对齐方式，这里的对齐指的是3D世界的坐标。枚举值见下方
默认值ARWorldAlignmentGravity
 */
@property (nonatomic, readwrite) ARWorldAlignment worldAlignment;

/**
是否需要自适应灯光效果，默认是YES
 */
@property (nonatomic, readwrite, getter=isLightEstimationEnabled) BOOL lightEstimationEnabled;

/**
是否需要提供音频，默认是NO
 */
@property (nonatomic, readwrite) BOOL providesAudioData;

@end


//世界会话追踪配置，苹果建议我们使用这个类，这个子类只有一个属性，也就是可以帮助我们追踪相机捕捉到的平地
@interface ARWorldTrackingSessionConfiguration : ARSessionConfiguration

/**
侦查类型。枚举值见下方（默认侦查平地）
 */
@property (nonatomic, readwrite) ARPlaneDetection planeDetection;

@end
```

//追踪对其方式，这个决定了会话的参考坐标系（参照物）.

```OC
typedef NS_ENUM(NSInteger, ARWorldAlignment) {
    /** Aligns the world with gravity that is defined by vector (0, -1, 0). */
    ARWorldAlignmentGravity,
    
    /** Aligns the world with gravity that is defined by the vector (0, -1, 0)
     and heading (w.r.t. True North) that is given by the vector (0, 0, -1). */
    ARWorldAlignmentGravityAndHeading,
    
    /** Aligns the world with the camera’s orientation. */
    ARWorldAlignmentCamera
};

typedef NS_OPTIONS(NSUInteger, ARPlaneDetection) {
    /** No plane detection is run. */
    ARPlaneDetectionNone        = 0,
    
    /** Plane detection determines horizontal planes in the scene. */
    ARPlaneDetectionHorizontal  = (1 << 0),
};
```

### (12) ARSKView

ARSKView也是AR视图，只不过他是2D的，由于2D比3D简单很多，并且ARSKView基本与ARSCNView类似，所以这里不做重复介绍。详细内容可参考ARSCNView.

# 参考

- [ARKit API介绍](https://www.jianshu.com/p/633c8ce07c83#2)
- [直击苹果 ARKit 技术](https://www.jianshu.com/p/7faa4a3af589)









