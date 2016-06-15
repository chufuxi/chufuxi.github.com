---
layout:     post
title:      Android UI Performance I
date:       2015-07-22 12:31:19
summary:    理解Display List,VSYNC，Android渲染过程
categories: performance
---


## Display List  
在3.0之前，Android平台是不提供硬件加速的功能的，所有绘制过程是通过软件绘制来完成，  
在3.0之后，Android提供的硬件加速的功能，就是我们所熟悉的下面配置
![hardwareacc](http://7xkehk.com1.z0.glb.clouddn.com/hardwareacc.png)  

使用硬件加速以后，Android Drawing Model发生了很大的变化

Software-based drawing model

1. Invalidate the heirarchy  
2. Draw the heirarchy  

当应用程序更新界面时，在相应需要更新的界面使用invalidate, 这个invalidate会向上回溯到整个界面的Root View处，再计算所有需要更新内容的区域
并且所有和这个内容区域有交叠，覆盖等影响的View都会被要求进行重绘，这样来完成整个界面的更新。

问题：
所有的交叠区域的View都会被重新绘制，也就是说所有View的draw都会被重新执行一边。
比方说一个最顶层界面的Button需要更新，将会导致覆盖在其下的所有的界面进行重新绘制。
考虑下如果View的onDraw比较复杂, 比如一些经过定制custom view，这将导致每次的界面更新都异常的耗时。

Hardware accelerated drawing modle

1. Invalidate the heirarchy  
2. Record and update display lists  
3. Draw the display lists

在Hardware accelerated drawing modle中，View.draw只会生成Display list，不再去完成整个绘制动作。当某个View产生更新时，实际上只需要更新这个View
的Displaylist，而其他的交叠区域的View不会更新它们的Display lists，而是继续复用之前的Display list进行绘制。
换句话来说，将只有需要更新View的draw会被调用来产生新的Display list，而其他的View将不会调用draw进行绘制。

除此之外，hardware drawing mode也提高了动画的绘制效率，View的Property Animation，包括alpha, rotateX/Y, translateX/Y,scaleX/Y等属性。 在做动画时，不需要
invalidate相应的View（不需要再draw一次）, 而只需要简单对包含这个View的Layer做相应的属性变化。
![dlprop](http://7xkehk.com1.z0.glb.clouddn.com/dlprop.png)

属性变化时不需要再重新生成DisplayList

## VSYNC

Android在JB版本就开始引入了VSYNC来完成刷新率和渲染速度的同步，用来解决Tearing问题，也就是同时在屏幕上看到新旧画面的问题。

![tearing](http://7xkehk.com1.z0.glb.clouddn.com/tearing.png)

因为GPU对画面处理是逐行进行处理的，当前一帧的图像显示结束以后，GPU开始逐行绘制新的画面，如果在这个绘制中间
把Framebuffer翻转到显示设备上就会出现这样的情况。


Android通过VSYNC来同步屏幕刷新率和GPU的帧处理速度，只会在VSYNC同步信号到来时进行FrameBuffer翻转，如果信号
到来时，GPU还没有完成画面绘制，就继续显示上一帧的画面

VSYNC在ISC版本之后开始用来同步所有的界面沪指相关的工作，来减少界面显示，动画的jank现象，增加平滑度。

![drawingWithoutVSYNC](http://7xkehk.com1.z0.glb.clouddn.com/drawingWithoutVSYNC.png)

我们可以简单把绘制工作看成两部分

1. CPU完成draw-产生Displaylist
2. GPU完成render-执行Displaylist的绘制命令

在未使用Vsync同步时，CPU的draw工作和GPU的rendering工作是，所以当本可以在一个Vsync周期中（16ms）完成的工作
被横跨在Vsync边界上时，就会导致无法在16ms内完成一帧的绘制工作，只能重复的显示上一帧的内容，从而产生Jank.

![drawingVSYNC](http://7xkehk.com1.z0.glb.clouddn.com/drawingVSYC.png)

使用VSYNC来作为所有系统中关联到界面绘制工作（rendering，animations...)的触发器，简单来看就是在每个VSYNC信号开始时
触发CPU，GPU进行绘制工作，保证能够在一VSYNC绘制周期内完成的工作不会横跨在边界上产生上图看到的jank现象。

保证所有的绘制工作在一个VSYNC周期中完成，16ms是完成所有绘制工作的底线。

### Android渲染过程

我们一般用来实现界面的布局文件被翻译成GPU可识别的样式，也就是GL命令和需要的资源，再通过GPU的按照命令绘制实际图形，最终再显示在设备屏幕，形成我们最终可见的界面。

![rendering](http://7xkehk.com1.z0.glb.clouddn.com/performance2/rendering.png)

一个Button简单绘制过程就如下图所示

![btn](http://7xkehk.com1.z0.glb.clouddn.com/performance2/btnRender.png)

这里最重要的部分就是创建DisplayList提供给GPU进行绘制。在Android的绘制模型中，对View.draw的执行实际上就是创建DisplayList的过程。

![btnfull](http://7xkehk.com1.z0.glb.clouddn.com/performance2/btnRenderFull.png)

**Invalidate**过程会导致整个界面的重新绘制，包括重新创建DisplayList（重新onDraw）

![btninvalidate](http://7xkehk.com1.z0.glb.clouddn.com/performance2/btnRenderInvalidate.png)

**Property Change**过程任然使用原先创建的DisplayList绘制，来进行矩阵变换达到属性变换的效果 （无需onDraw）

![btnproperty](http://7xkehk.com1.z0.glb.clouddn.com/performance2/btnRenderProperty.png)
