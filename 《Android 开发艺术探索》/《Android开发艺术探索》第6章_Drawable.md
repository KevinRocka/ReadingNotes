#Chapter 6.Drawable

Time : 2016.10.9

Author : Rocka 

## Drawable简介
* Android的Drawable表示的是一种可以在Canvas上进行绘制的概念，它的种类很多，最常见的就是图片和颜色了。它有两个重要的优点：一是比自定义view要简单；二是非图片类型的drawable占用空间小，利于减小apk大小。

* Drawable是抽象类，是所有Drawable对象的基类。

* Drawable的内部宽/高可以通过getIntrinsicWidth和getIntrinsicHeight方法获取，但是并不是所有Drawable都有内部宽/高。图片Drawable的内部宽高就是图片的宽高，但是颜色Drawable就没有宽高的概念，它一般是作为view的背景，所以会去适应view的大小，这两个方法都是返回-1。

## Drawable分类
* BitmapDrawable表示一张图片，对应xml标签

	```
	<?xml version="1.0" encoding="utf-8"?>
<bitmap / nine-patch
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    android:src="@[package:]drawable/drawable_resource"
	    android:antialias=["true" | "false"]
	    android:dither=["true" | "false"]
	    android:filter=["true" | "false"]
	    android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
	                      "fill_vertical" | "center_horizontal" | "fill_horizontal" |
	                      "center" | "fill" | "clip_vertical" | "clip_horizontal"]
	    android:tileMode=["disabled" | "clamp" | "repeat" | "mirror"] />
	```
	* android:antialias：是否开启图片抗锯齿功能。开启后会让图片变得平滑，同时也会一定程度上降低图片的清晰度，建议开启；
	* android:dither：是否开启抖动效果。当图片的像素配置和手机屏幕像素配置不一致时，开启这个选项可以让高质量的图片在低质量的屏幕上还能保持较好的显示效果，建议开启。
	* android:filter：是否开启过滤效果。当图片尺寸被拉伸或压缩时，开启过滤效果可以保持较好的显示效果，建议开启；
	* android:gravity：当图片小于容器的尺寸时，设置此选项可以对图片进行定位。
	* android:tileMode：平铺模式，有四种选项["disabled" | "clamp" | "repeat" | "mirror"]。当开启平铺模式后，gravity属性会被忽略。repeat是指水平和竖直方向上的平铺效果；mirror是指在水平和竖直方向上的镜面投影效果；clamp是指图片四周的像素会扩展到周围区域，这个比较特别。
* NinePatchDrawable，表示一张.9图， ,在bitmap标签中也可以使用.9图。
* ShapDrawable表示以颜色构造的图形，可以纯色，也可以渐变。

	```
	<?xml version="1.0" encoding="utf-8"?>
<shape    
    xmlns:android="http://schemas.android.com/apk/res/android"    
    android:shape=["rectangle" | "oval" | "line" | "ring"] >    
    	<corners        //当shape为rectangle时使用
	        android:radius="integer"        //半径值会被后面的单个半径属性覆盖，默认为1dp
	        android:topLeftRadius="integer"        
	        android:topRightRadius="integer"        
	        android:bottomLeftRadius="integer"        
	        android:bottomRightRadius="integer" />    
	    <gradient       //渐变
	        android:angle="integer"        
	        android:centerX="integer"        
	        android:centerY="integer"        
	        android:centerColor="integer"        
	        android:endColor="color"        
	        android:gradientRadius="integer"        
	        android:startColor="color"        
	        android:type=["linear" | "radial" | "sweep"]        
	        android:useLevel=["true" | "false"] />    
	    <padding        //内边距
	        android:left="integer"        
	        android:top="integer"        
	        android:right="integer"        
	        android:bottom="integer" />    
	    <size           //指定大小，一般用在imageview配合scaleType属性使用
	        android:width="integer"        
	        android:height="integer" />    
	    <solid          //填充颜色
	        android:color="color" />    
	   	<stroke         //边框
	      	android:width="integer"        
	        android:color="color"        
	        android:dashWidth="integer"        
	        android:dashGap="integer" />
</shape>
	```
	* android:shape：默认的shape是矩形，line和ring这两种形状需要通过<stroke>来制定线的宽度和颜色，否则看不到效果。
	* gradient：solid表示纯色填充，而gradient表示渐变效果。andoid:angle指渐变的角度，默认为0，其值必须是45的倍数，0表示从左到右，90表示从下到上，其他类推。
	* padding：这个表示的是包含它的view的空白，四个属性分别表示四个方向上的padding值。
size：ShapeDrawable默认情况下是没有宽高的概念的，但是可以如果指定了size，那么这个时候shape就有了所谓的固有宽高，但是作为view的背景时，shape还是会被拉伸或者缩小为view的大小。
* LayerDrawable 对应标签<layer-list>，表示层次化的Drawable集合，实现一种叠加后的效果, 属性android:top/left/right/bottom表示drawable相对于view的上下左右的偏移量，单位为像素。
	
* StateListDrawable对应 ,可以根据view的状态自动选择drawable,对应标签<selector>，也是表示Drawable集合，每个drawable对应着view的一种状态。

* LevelListDrawable，对应标签<level-list>，同样是Drawable集合，每个drawable还有一个level值，根据不同的level，LevelListDrawable会切换不同的Drawable，level值范围从0到100000。
* TransitionDrawable标签<transition>，用于实现两个Drawable之间的淡入淡出效果。
	
	```
	<transition xmlns:android="http://schemas.android.com/apk/res/	android" >
    <item android:drawable="@drawable/shape_drawable_gradient_linear"/>
    <item android:drawable="@drawable/shape_drawable_gradient_radius"/>
</transition>
TransitionDrawable drawable = (TransitionDrawable) v.getBackground();
drawable.startTransition(5000);
	```
* InsetDrawable标签<inset>，它可以将其他drawable内嵌到自己当中，并可以在四周留出一定的间距。当一个view希望自己的背景比自己的实际区域小的时候，可以采用InsetDrawable来实现。

	```
	<inset xmlns:android="http://schemas.android.com/apk/res/android"
    android:insetBottom="15dp"
    android:insetLeft="15dp"
    android:insetRight="15dp"
    android:insetTop="15dp" >

    <shape android:shape="rectangle" >
        <solid android:color="#ff0000" />
    </shape>
	</inset>
	```
* ScaleDrawable可以根据level将指定的drawable缩放到一定比例。Level越大，内部drawable看起来也就越大

* ClipDrawable可以根据自己当前level裁剪另一个drawable。Level越大，裁剪区域越小

	```
	<clip xmlns:android="http://schemas.android.com/apk/res/android"
    	android:clipOrientation="vertical"
    	android:drawable="@drawable/image1"
    	android:gravity="bottom" />
	```

* AnimationDrawable可以播放帧动画 
* ColorDrawable代表单色

## Drawable自定义

* Drawable的工作核心就是draw方法，所以自定义drawable就是重写draw方法，当然还有setAlpha、setColorFilter和getOpacity这几个方法。当自定义Drawable有固有大小的时候最好重写getIntrinsicWidth和getIntrinsicHeight方法。
* Drawable的内部大小不等于Drawable的实际区域大小，Drawable的实际区域大小可以通过它的getBounds方法来得到，一般来说它和view的尺寸相同。