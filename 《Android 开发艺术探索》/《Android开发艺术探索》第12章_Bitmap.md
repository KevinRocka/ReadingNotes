#Chapter 12.Bitmap

Time : 2016.8.15 - 2016.8.15

Author : Rocka

## Bitmap高效加载

* BitmapFactory类提供了四类方法：decodeFile、decodeResource、decodeStream和decodeByteArray从不同来源加载出一个Bitmap对象，最终的实现是在底层实现的。
* 采用BitmapFactory.Options按照一定的采样率来加载所需尺寸的图片，因为imageview所需的图片大小往往小于图片的原始尺寸
* BitmapFactory.Options的inSampleSize参数，即采样率官方文档指出采样率的取值应该是2的指数，例如k，那么采样后的图片宽高均为原图片大小的 1/k。inSampleSize为2，那么采样后的图片其宽/高均为原图大小的1/2，而像素为原图的1/4，内存也为原图的1/4。
* 如何获取图片的采样率呢? 
	* 将BitmapFactory.Options的inJustDecodeBounds参数设置为true并加载图片
	* 从BitmapFactory.Options中取出原始的宽高信息，它们对应于outWidth和outHeight
	* 根据采样率的规则并结合目标View的所需大小计算出采样率inSampleSize
	* 将BitmapFactory.Options的inJustDecodeBounds参数设为false，然后重新加载图片 
* BitmapFactory只会解析图片的原始宽高信息，并不会去真正地加载图片，所以这个操作是轻量级的,需要注意的是，这个时候BitmapFactory获取的图片宽高信息和图片的位置以及程序运行的设备有关，这都会导致BitmapFactory获取到不同的结果。

## Bitmap缓存策略
> 最常用的缓存算法是LRU，核心是当缓存满时，会优先淘汰那些近期最少使用的缓存对象，系统中采用LRU算法的缓存有两种：LruCache(内存缓存)和DiskLruCache(磁盘缓存)。

* 几种引用
	* 强引用(StrongReference)：直接对象引用
	* 软引用(SoftReference)：当一个对象只有软引用存在时，系统内存不足对象会被gc回收
	* 弱引用(WeakReference)：当一个对象只有弱引用存在时，对象随时会被系统回收
* LruCache是Android 3.1才有的，通过support-v4兼容包可以兼容到早期的Android版本。LruCache类是一个线程安全的泛型类，它内部采用一个LinkedHashMap以强引用的方式存储外界的缓存对象，其提供了get和put方法来完成缓存的获取和添加操作，当缓存满时，LruCache会移除较早使用的缓存对象，然后再添加新的缓存对象。
* DiskLruCache磁盘缓存，它不属于Android sdk的一部分，DiskLruCache的创建、缓存查找和缓存添加操作

## ImageLoader的使用
* 避免发生列表item错位的解决方法
	* 给显示图片的imageview添加tag属性，值为要加载的图片的目标url，显示的时候判断一下url是否匹配。
* 化列表的卡顿现象
	* 不要在getView中执行耗时操作，不要在getView中直接加载图片，否则肯定会导致卡顿；
	* 控制异步任务的执行频率：在列表滑动的时候停止加载图片，等列表停下来以后再加载图片；
	* 使用硬件加速来解决莫名的卡顿问题，给Activity添加配置android:hardwareAccelerated="true"；
	* 解决大范围上下滑动,异步任务会造成线程池的拥堵并随即带来大量的UI更新操作，解决办法是在OnScrollListener的OnScrollStateChanged方法中判断是否滑动，停止滑动在加载




	