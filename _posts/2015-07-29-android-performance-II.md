---
 layout:     post
 title:      Android UI Performance II
 date:       2015-08-29 23:50:00
 summary:    OverDraw，View Hierarchy，Alpha问题
 categories: performance
---

### 达成60FPS
视觉暂留是产生动画的基础，一般来说人眼的视觉暂留大约是1/10~1/16s，也就是说当动画速度达到15FPS，就能够在人脑中形成连续的动画现象。  
当然这个帧率还是不够的，也就相当于翻书的速度。这样的画面会产生明显的卡顿感。那么当帧率到达24帧/30帧，也就是现在大多数电影/电视的帧率，  
已经能够达到流畅显示的影像的程度了。当然电影/电视的同量级的帧率可不能和应用UI的帧率同日而语哟，即时达到了30FPS，界面也会表现的不忍直视。  
达到60FPS才能够享受到流畅，顺滑的体验。超过60FPS的帧率，人眼的感知度有限，基本浪费了


### 2.5 screen content

保证绘制内容在2.5个屏幕大小以内，也就是说Android平台可以保证在16ms完成的绘制内容就是2.5个屏幕大小，超出这个限制，绘制更多的内容就会产生延迟。

但是在实际的中，高分低能手机，复杂的界面结构，多层叠加的界面都会隐形的限制实际能够绘制的内容。

尽量减少实际的绘制内容，特别是控制好对图片的绘制，通过调整图片的尺寸，采样率，RGB格式都可以帮助减少实际的绘制内容。

### OverDraw

OverDraw是界面中比较常见会发生的问题

想象屏幕就是一块画布，在这个画布上一层一层的刷上不同颜色的油漆。问题就来了，除了顶层颜色的油漆，下层的油漆是不会被看见的，白白浪费了刷油漆的时间，这就是OverDraw问题。

![overdraw](http://7xkehk.com1.z0.glb.clouddn.com/performance2/overdraw.png)

哪里会发生OverDraw问题那？

Background！Background！Background！

- 多层界面中每层的背景
- TextView，ImageView的背景
- OverLap的图形

OverDraw Tools

工具使用： [OverDraw Debuger](https://developer.android.com/tools/performance/debug-gpu-overdraw/index.html)

大面积的红色就表示了OverDraw的问题，所以我们的目标就是变身为右侧的蓝色状态！
![overdrawdebuger](http://7xkehk.com1.z0.glb.clouddn.com/performance2/overdrawdebugger.png)

大面积的OverDraw往往会导致性能问题。查看界面层次，去除根本不会在屏幕上展示的背景，图片，尽可能减少OverDraw的面积！

值得考虑是，在App的设计阶段，往往会更注重界面的美观，交互的顺畅，对是否影响程序性能考虑的不多，但是各种绚丽图片，渐变效果，虚无的透明度，多层的动画都对性能有非常高的要求，这也是导致为啥效果图和实际实现的效果总是有差距的原因。所以在设计阶段，就需要研发，设计在一起来综合考虑App的界面，交互以及对性能的要求！

### View Hierarchy

View Hierarchy的问题就是在整个界面层次中加入了多余的不需要的不需要的节点。

问题

- 单节点组成的链条
- LinearLayout控制布局
- ImageView+TextView的组件
- 复杂的组件（如需要多个ImageView,TextView进行展示的条目）

对策

- 尽量减少界面布局中的单节点链，使用merge标签
- 慎用LinearLayout进行布局，尽量使用RelativeLayout

>在LinearLayout中使用weight会使界面多次进行measure来确认View所需的尺寸，慎用！

- 尽量使用Android自有的View组件，比如TextView中可以设置CompoundDrawables来展示图片
- 复杂，高复用的组件，考虑使用自定义View来实现

>对于固定位置布局结构，可以通过自定义View来实现一次measure。RelativeLayout则需要两次meausre

工具使用： [Hierarchy Viewer](http://developer.android.com/tools/performance/hierarchy-viewer/index.html)

### Alpha

现在的很多界面都采用了带有透明度的背景,图片，使用fade in/out动画，这些都涉及到了界面的Alpha。

Alpha有时会产生比较麻烦的性能问题，特别是在一些硬件加速支持不好的情况下。

在使用View.setAlpha时，一般情况下会在offscreen-buffer中重新进行绘制，对于软件渲染来说就是重新构建一张新的Bitmap，
对于硬件渲染来说就是构建OpenGL texture/layer/FBO，来组建这个View上的所有图层。这样就产生了一次额外的绘制合成，加倍了绘制速度。

对策

- 小心使用setAlpha，特别是长时间，频繁的使用。
- 如果一定要使用的话，请考虑使用View.setLayerType设置LAYER\_TYPE\_HARDWARE

>这会保存当前的Layer，复用这个Layer来进行属性变换，较少重复绘制。
要注意的是，在使用这种方法的同时也要确保Layer中内容是不变的只是在做属性变换。否则会先重新绘制完，再保存Layer。
这样就白白浪费了GPU的资源，来反复的保存Layer，还达不到加速的效果。

- 遇到View出现奇怪的问题时，请尝试关闭硬件加速，使用软件渲染来试试。
