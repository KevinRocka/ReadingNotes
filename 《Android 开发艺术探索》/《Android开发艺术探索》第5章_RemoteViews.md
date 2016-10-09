#Chapter 5.RemoteViews

Time : 2016.8.13

Author : Rocka 

## RemoteView应用
* 跨进程更新界面，widget和notification都运行在SystemServer进程中

* 单击事件一般通过PendingIntent发广播的方式来实现

* onEanble()只在第一次添加回调，onDeleted()删除一次回调一次，onDisable()最后一个删除时回调，onUpdate()每次添加或更新时回调

## PendingIntent

* PendingIntent是即刻发生，Intent是立刻发生.典型场景还是RemoteViews的单机事件

* PendingIntent支持三种待定意图：启动Activity(getActivity)，启动Service(getService)，发送广播(getBroadcast)

* PendingIntent 的requestCode相同并且Intent也相同，则证明PendingIntent相同，Intent相同则需要ComponentName和intent-filter相同。

* PendingIntent 中flags
   * FLAG_ONE_SHOT:同类通知只能使用一次，后续单机打开无效
   * FLAG_NO_CREATE:没意义
   * FLAG_CANCEL_CURRENT: 如果PendingIntent已经存在，都会被cancle，系统创建一个新的，那些被cancel了的将无法打开
   * FLAG_UPDATE_CURRENT: 如果PendingIntent已经存在，它们都会被更新，Intent中的Extras被替换成最新的

* 分析NotificationManager.nofify(id, notification) [未测试，看着有点晕]

	* 如果参数id是常量，那么多次调用notify只能弹出一个通知，后续的通知会把前面的通知完全替代掉；

	* 如果参数id每次都不同，那么当PendingIntent不匹配的时候，不管采用何种标志位，这些通知之间不会相互干扰；

	* 如果参数id每次都不同，且PendingIntent匹配的时候，那就要看标志位：



## RemoteView内部机制
* 通知栏和widget布局文件实际上是在NotificationManagerService和AppWidgetService被加载，运行在SystemServer进程中，系统首先将view操作封装到action对象中，并将这些对象跨进程传输到远程进程，远程进程通过RemoteViews的apply方法进行View的更新操作，apply方法会遍历所有action对象来调用它们的apply方法

* 无法使用自定义View，EditText不支持在RemoteView中使用

* RemoteViews实现了Parcelable接口，它会通过Binder传递到SystemServer进程，系统会根据RemoteViews中的包名信息获取到应用中的资源，从而完成布局文件的加载

* 系统将view操作封装成Action对象，Action同样实现了Parcelable接口，通过Binder传递到SystemServer进程。远程进程通过RemoteViews的apply方法来进行view的更新操作，RemoteViews的apply方法内部则会去遍历所有的action对象并调用它们的apply方法来进行view的更新操作。这样做的好处是不需要定义大量的Binder接口，其次批量执行RemoteViews中的更新操作提高了程序性能。

* RemoteView的大部分方法都是通过set反射完成

* RemoteView中，apply加载布局更新界面，reApply只会更新界面

* setOnClickPendingIntent用于普通view设置单机事件，setPendingIntentTemplate和setOnClickFillInIntent组合使用用于ListView的item点击事件

## RemoteView特例

* 在两个应用中，用广播的方式完成跨进程通信，通过intent传递remoteviews，来更新另外一个进程的UI，当然RemoteView只能支持一些简单的view。