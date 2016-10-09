#Chapter 4.View_Principle

Time : 2016.8.11 - 2016.8.13

Author : Rocka 

## ViewRoot和DecorView

* ViewRoot 对应于ViewRootImpl，连接的WindowManager和DecorView的纽带，View 三大流程均是通过ViewRoot来完成，ActivityThread中，当Activity对象创建完毕后，将DecorView添加到Window中，同时会创建ViewRootImpl对象，并将ViewRootImpl对象和DecorView关联
* DecorView顶级View，一般情况下包含一个垂直方向的LinearLayout，setContentView布局的确加到了id为content的FrameLayout,DecorView其实是一个FrameLayout，View层的事件先经过DecorView，然后传递我们的View

## MeasureSpec
* MeasureSpec 和 LayoutParams对应关系：
	* 三类specMode
		* UNSPECIFIED：基本无视；父容器不对View有任何限制，要多大给多大；一般用于系统内部用于测量
		* EXACTLY：父容器已经检测出View所需要的大小，View的大小就是SpecSize的值，它对应于LayoutParams中的match_parent和具体的数值
		* AT_MOST：父容器指定了一个可用大小即SpecSize，对应于LayoutParams中的warp_content
   	* View测量的时候，系统会将LayoutParams在父容器的约束下转换成对应的MeasureSpec，然后根据这个MeasureSpec来确定View测量后的宽高
   	* MeasureSpec不是唯一由LayoutParams来决定的，LayoutParams需要和父容器一起才能决定View的MeasureSpec，从而进一步确定view的宽高，对于DecorView，它的MeasureSpec由串口的尺寸和其自身的LayoutParams来决定的；对于普通View，它的MeasureSpec由父容器的MeasureSpec和自身的LayoutParams来共同决定
* 普通View的MeasureSpec创建规则：
	* 当View采取固定宽高时，不管父容器MeasureSpec是什么，View的MeasureSpec都是精确模式，并且大小是LayoutParams中的大小
	* 当View的宽高是match_parent时，如果父容器模式是精确模式，那么View的模式也是精确模式，大小是父容器的剩余空间，如果父容器是最大模式，那么view也是最大模式，大小不超过父容器剩余空间
	* View的宽高是wrap_content时，不管父容器的模式是精确模式还是最大模式，View的模式总是最大模式，大小不超过父容器剩余空间

## View的工作原理
- measure过程
	* getSuggestedMinimumWidth，如果没有设置背景，返回android:minWidth的值，可以为0；View设置了背景，返回android:minWidth和背景最小宽度两证的最大值。getSuggestedMinimumWidth/Height返回值就是View在UNSPECIFIED情况下的测量宽高
	* 获取View宽高方式：
		* Activity/View 的onWindowFocusChanged()获取焦点或者焦点改变都要调用，onPause(),onResume()执行也要多次调用
		* View.post(runnable)
		* ViewTreeObserver的OnGlobalLayoutListener接口，也要注意多次调用，记得removeListener()
		* 通过View.measure(int widthMeasureSpec,int heightMeasureSpec),不推荐
- layout过程
	* 一般在onLayout()中去获取测量的宽高，得到的才是准确的
	* getMeasureWidth()和getWidth()区别：最终值是相等的，第一个形成于measure过程，第二个形成于layout过程，两者赋值时机不同
- draw过程
	* 绘制背景background.draw(canvas);
	* 绘制自己ondraw();
	* 绘制子ViewdispatchDraw();
	* 绘制装饰onDrawScrollBars();

## 自定义View

* 让View支持wrap_content
* 让View支持padding
* View本身提供post方法，最好不要用handler
* 在onDetachedFromWindow()方法回调中停止线程和动画
* 注意viewgroup的padding以及子元素的margin
* View带有滑动嵌套的情况要处理好滑动冲突	