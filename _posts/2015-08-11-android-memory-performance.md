---
layout:     post
title:      Android Memory Performance
date:       2015-08-11 12:31:19
summary:    合理使用内存，避免泄露！
categories: performance
---

###内存使用对性能的影响

当使用内存时看起来是这样的！

~~~java
new ArraryList()
~~~

如果当前在Heap中有足够的空闲空间的话，就会分配好一块供ArraryList使用。

但是，如果恰好Heap中的空闲内存不够分配，就会触发回收来清理不再使用的内存，以满足内存使用的需求。

那么看起来就是这个样子。


![memReq](http://7xkehk.com1.z0.glb.clouddn.com/performance/mem/intro/mem_recl.png)

看上去垃圾回收机制聪明的帮我们解决了内存需求！ So far, so well!

那么哪里会有问题？

实际上Garbage Collection不是无代价，无消耗完成的！在Dalvik时代会GC会Pause应用程序所有线程，来进行标记，查找，收集等等内存回收的工作，等回收工作完成以后，才会恢复应用程序运行。

在最新的ART虚拟机上，即使有专门的后台线程来处理垃圾回收工作，任然会在回收结束时产生一个非常短暂的Pase。

所以，实际情况是这样的。

![dalvikGC](http://7xkehk.com1.z0.glb.clouddn.com/performance/mem/intro/dalvik_gc.png)

![artGC] (http://7xkehk.com1.z0.glb.clouddn.com/performance/mem/intro/art_gc.png)

如果在应用程序的Draw过程中引发了GC，会怎么样那？

![gcDraw](http://7xkehk.com1.z0.glb.clouddn.com/performance/mem/intro/gc_draw.png)

如果GC引发的Pause时间再长一点点...

![gcJunk](http://7xkehk.com1.z0.glb.clouddn.com/performance/mem/intro/junk.png)

Woops, 第二帧的绘制过程超过了16ms, 在这个时候应用就会表现为掉帧，如果有更多的帧在绘制过程产生这样的问题，用户就会感觉到明显的延迟，卡顿。

在使用ART虚拟机的设备上，GC引发的Pause的时间会比Dalvik上少很多，甚至只有几个毫秒的Pause时间。但是，即使是只有2个毫秒的Pause，也任然有可能使得帧绘制时间超过16ms的边界，引发效率问题。


###管理应用内存

**不要过度加载**

在使用图片时，根据实际需要来进行加载。用多少，加载多少，按照View的实际尺寸来裁剪图片。

从应用场景的角度来决定到底要如何来进行图片加载，越高质量的图片会要求更多的内存空间，平衡好图片质量和内存使用。

当应用展示图片时，RGB565和ARGB888格式是否有很大的区别？如果没有明显的区别，为何不使用能节省一半的格式。比如只是在展示一个40x40的头像。

**复用，复用**

使用大量的Object或者Bitmap，频繁的构建新的对象，会导致GC不得不去回收之前的大量使用过得内存。

使用Object Pool，Bitmap Pool来进行缓存，避免对象的重新创建，申请内存。

**避免内存泄露**

内存泄露会在不动声色中吃掉所有的空间，导致无内存可用，每次都可能去Alloc新的Heap空间，最终导致OOM。

在内存泄露中，特别要注意Context和View的泄露。

- 协调好不同的生命周期，确保在Context的生命周期结束时，关闭所关联**线程，文件，数据库**

- 谨慎使用生命周期外内部类，内部类会隐式含有外部类的引用，如果这内部类有很长生命周期就会导致本该被销毁的外部类无法被销毁。

~~~java
public LeakActivity extande Activity {

	//Leak activity if handler run out of this activity.
	private Handler mHandler = new Handler () {

		@Override
		public void handlerMessage(Message msg) {
			//do something
		}
	}
}
~~~

**不要使用Enums**

Enums需要的内存是使用static constants的两倍！

使用@IntDef来替代Enums

[https://developer.android.com/reference/android/support/annotation/IntDef.html](https://developer.android.com/reference/android/support/annotation/IntDef.html)
[http://tools.android.com/tech-docs/support-annotations](http://tools.android.com/tech-docs/support-annotations)
