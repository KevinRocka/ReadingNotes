#Chapter 7.Animations

Time : 2016.8.14

Author : Rocka 

## View动画
* 建议采用XML定义动画，比代码可读性更好
* android:fillAfter 动画结束以后是否停在结束位置
* 自定义View动画需要继承Animation，重写initialize和applyTransformation，在后者进行矩阵变换即可
* 帧动画可以通过AnimationDrawable来使用帧动画，也可以通过Handler发消息来替换存在SoftReference里面的ImageView

## View动画特殊场景
* 使用LayoutAnimation来给ViewGroup的每个item加上出场效果
* overridingPendingTransition()来改变activity切换的场景，必须在startactivity()和finish()后面调用。Fragment的切换用FragmentTransaction的setCustomAnimations()

## 属性动画
* nineoldandroids动画库在api11以前内部是通过代理View动画来实现的,基本支持所有android系统
* 不建议使用xml来实现属性动画，因为在xml中有时候不知道屏幕宽度等
* Interpolator插值器主要来改变动画变化率，TypeEvaluator类型估值器，根据当前属性改变的百分比来计算改变后的属性值
* AnimatorListenerAdapter()有常用回调，AnimatorUpdateListener监听整个动画过程，没播放一帧，回调一次
* 对object属性abc做动画条件：
	* 1.object必须提供getAbc()和setAbc()方法
	* object的setAbc()做的改变通过某种方法UI会改变
* 属性动画不生效解决办法
	* 有权限的话，给你对象加上set和get方法
	* 用一个类来包装原生对象，间接为其提供get和set方法
	* 采用ValueAnimator，监听动画过程，自己实现属性改变
* 属性动画必须运行在有Looper的线程中
* 属性动画的核心还是通过反射给每一帧的动画设置属性值


## 动画注意事项

* OOM，尽量避免使用帧动画
* 属性动画有一类无限循环动画需要在Activity退出时及时停止，View动画不存在这问题
* View动画是对View的影像动画，需要调用clearAnimation()清楚View动画即可
* 尽量使用DP
* 使用动画的过程中，建议开启硬件加速
* 3.0以前不管是View动画还是属性动画，View移动后的事件都无法点击老位置可以.3.0以后,属性动画单机事件随位置移动而移动