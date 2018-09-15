---
title: iOS开发-传感器坐标系问题
date: 2018-09-12 18:14:32
categories:
- 编程
- iOS
tags:
- 编程
- iOS
copyright:
---

<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

# 传感器坐标系

移动设备内置的传感器采集得到的数据均源自传感器本体坐标系下。然而，在现实应用中，常常有如下需求：求取传感器数据在世界坐标系下的数据。例如，导航应用。

常见的手机本体坐标系如下图所示：

![](http://p55se4hrx.bkt.clouddn.com/images/programmings/iOS/iOS%E5%BC%80%E5%8F%91_%E6%89%8B%E6%9C%BA%E5%9D%90%E6%A0%87%E7%B3%BB.png)

世界坐标系如下图所示：

![](http://p55se4hrx.bkt.clouddn.com/images/programmings/iOS/iOS%E5%BC%80%E5%8F%91_%E4%B8%96%E7%95%8C%E5%9D%90%E6%A0%87%E7%B3%BB.png)

为了区分与手机坐标系的区别，这里把地球坐标系加一个下标w，在此坐标系中：

zw 垂直与地面向上
yw与zw垂直，指向北。
xw垂直与zw与yw确定的平面，指向东。

# 加速度计

iOS的加速度传感器能够提供很多有用的数据，比如rotationMatrix和指向三轴方向的分量x、y、z。iOS SDK提供的传感器管理类CMDeviceMotion用来获取大多数的传感器数据，对于加速度来说，主要是userAccelerate分量数据。

```Swift
import CoreMotion
var motionManager = CMMotionManager()
...
motionManager.startDeviceMotionUpdatesUsingReferenceFrame(CMAttitudeReferenceFrameXTrueNorthZVertical, toQueue: NSOperationQueue.currentQueue(), withHandler: {
    ...
    var acc: CMAcceleration = deviceMotion.userAcceleration
    var rot = deviceMotion.attitude.rotationMatrix
    self.ax = (acc.x*rot.m11 + acc.y*rot.m21 + acc.z*rot.m31)*9.81
    self.ay = (acc.x*rot.m12 + acc.y*rot.m22 + acc.z*rot.m32)*9.81
    self.az = (acc.x*rot.m13 + acc.y*rot.m23 + acc.z*rot.m33)*9.81
})
```

使用startDeviceMotionUpdatesUsingReferenceFrame方法可以设置参考系，这个参考系只对CMDeviceMotion中的attitude变量有效。也就是说在Block中得到的attitude是相对于设定的参考系的，而参考系的设定可以根据自己的需求来进行，通常有一下集中可设定的值：

```OC

/*
 *  CMAttitudeReferenceFrame
 *  
 *  Discussion:
 *    CMAttitudeReferenceFrame indicates the reference frame from which all CMAttitude
 *        samples are referenced.
 *
 *    Definitions of each reference frame is as follows:
 *        - CMAttitudeReferenceFrameXArbitraryZVertical describes a reference frame in
 *          which the Z axis is vertical and the X axis points in an arbitrary direction
 *          in the horizontal plane.
 *        - CMAttitudeReferenceFrameXArbitraryCorrectedZVertical describes the same reference
 *          frame as CMAttitudeReferenceFrameXArbitraryZVertical with the following exception:
 *          when available and calibrated, the magnetometer will be used to correct for accumulated
 *          yaw errors. The downside of using this over CMAttitudeReferenceFrameXArbitraryZVertical
 *          is increased CPU usage.
 *        - CMAttitudeReferenceFrameXMagneticNorthZVertical describes a reference frame
 *          in which the Z axis is vertical and the X axis points toward magnetic north.
 *          Note that using this reference frame may require device movement to 
 *          calibrate the magnetometer.
 *        - CMAttitudeReferenceFrameXTrueNorthZVertical describes a reference frame in
 *          which the Z axis is vertical and the X axis points toward true north.
 *          Note that using this reference frame may require device movement to 
 *          calibrate the magnetometer.
 */
typedef NS_OPTIONS(NSUInteger, CMAttitudeReferenceFrame) __TVOS_PROHIBITED {
	CMAttitudeReferenceFrameXArbitraryZVertical = 1 << 0,
	CMAttitudeReferenceFrameXArbitraryCorrectedZVertical = 1 << 1,
	CMAttitudeReferenceFrameXMagneticNorthZVertical = 1 << 2,
	CMAttitudeReferenceFrameXTrueNorthZVertical = 1 << 3
};
```

在上面获取数据的实例中，使用的是x轴指向地球北极，z轴指向地心，也就是地理上的地心方向。

通过attitude得到一个旋转向量rot，然后只要将原来相对于设备的参考系向量旋转到世界参考系中就可以了。

这个旋转使用了矩阵v_world = M^-1 * v_device = M^T * v_device。这里的M就是旋转矩阵rot，因为矩阵旋转的特性，一个矩阵的转置矩阵和逆矩阵是相等的。

而在计算的使用分量都乘以了一个9.81，其实这个就是物理上的g，所有的加速度都是以g为单位的。

# 坐标系转换

![](http://p55se4hrx.bkt.clouddn.com/images/programmings/iOS/iOS%E5%BC%80%E5%8F%91_%E5%9D%90%E6%A0%87%E7%B3%BB%E8%BD%AC%E6%8D%A2.png)

# 参考

- [如何获取iOS世界坐标系中的加速度 --- 关于坐标系的转换](https://robinchao.github.io/2016/09/motion-relative-to-world/)
- [安卓坐标系转换之一：从手机坐标系到地球坐标系](https://blog.csdn.net/yq_forever/article/details/79511676)