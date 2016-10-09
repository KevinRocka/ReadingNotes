#Chapter 1.Activitys	

Time : 2016.7.25 10:42

Author : Rocka 



##LifeCyle生命周期

-  正常情况
	* onCreate --> onStart --> onResume --> 运行--> 按返回键结束程序--> onPause-->onStop-->onDestory
	* Activity切换时，旧Activity的onPause会先执行，然后才会启动新的Activity
	* Activity在异常情况下被回收时，onSaveInstanceState方法会被回调，回调时机是在onStop之前，当Activity被重新创建的时 候，onRestoreInstanceState方法会被回调，时序在onStart之后；
* 异常情况
	* 资源相关的系统配置发生改变 ：像横竖屏切换，Activity销毁并重建，onPause,onStop,onDestory均会调用，系统调用onSaveInstanceState来保存当前Activity状态。这个方法是在onStop之前，当Activity重建的时候系统会把onSaveInstanceState所保存的Bundle作为对象传递给onRestoreInstanceState和onCreate方法。
	* 资源内存不足导致低优先级Activity被杀死 : 系统内存不足时，会按照以上顺序杀死Activity，并通过onSaveInstanceState和onRestoreInstanceState这两个方法来存储和恢复数据。




##LaunchMode启动模式
- 启动模式
 * standard ：标准模式。每次启动会重新创建新的实例，谁启动了这个Activity，这个Activity就在谁的栈里。
 * singleTop ：栈顶复用模式。新Activity位于栈顶，Activity不会被重新创建，该Activity的onNewIntent方法会被回调，onCreate和onStart并不会被调用。
 * singleTask ：栈内复用模式(单实例模式)。只要该Activity在一个栈中存在，都不会重新创建，onNewIntent会被回调。如果不存在，系统会先寻找是否存在需要的栈，如果不存在该栈，就创建一个任务栈，然后把这个Activity放进去；如果存在，就会创建到已经存在的这个栈中。(D的任务栈为S1，S1的任务栈为ADBC)，singleTask默认具有cleartTop的效果，会导致D上面所有的出站，最终为AD。
 * singleInstance ：加强单实例模式。具有此种模式的Activity只能单独存在于一个任务栈。
* TaskAffinity ：任务相关性，不能和包名相同，一般主要和singleTask启动模式或者allowTaskReparenting配对使用.
* Flags：FLAG\_ACTIVITY\_CLEAR\_TOP一般和singleTask启动模式一起出现。同一栈中为与它上Activity都要出栈。



##IntentFilter匹配规则

-  匹配规则
 * action匹配规则 : 可以有多个action，要求intent中的action存在且必须和过滤规则中的其中一个相同区分大小写；
 * category匹配规则 : 系统会默认加上一个android.intent.category.DEAFAULT，所以intent中可以不存在category，但如果存在就必须匹配其中一个；
 * data匹配规则 : 要求和action相似，如果规则定义data，Intent也必须定义可以匹配的。

* data : data由两部分组成，mineType和URI。mineType类型，比如image/jpeg、audio/mpeg4-generic和video/*
* URI : [scheme]://[host]:[port]/[path]- tips:path中“\*”要写成“\\\\*”,"\\"要写成“\\\\\\\\”
* 如果要为Intent指定完整的data，必须要调用setDataAndType方法，不能分别调用setData和setType因为会分别至空值
* 通过隐式方法启动Activity的时候一定要判断空，看是否有Activity匹配我们的Intent，判断方法为：PackageManager的resolveActivity或者Intent的resolveActivity.

 