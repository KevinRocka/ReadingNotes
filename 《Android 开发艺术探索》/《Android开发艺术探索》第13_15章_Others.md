#Chapter 13-15.Others

Time : 2016.8.15 - 2016.8.15

Author : Rocka

## CrashHandler
* setDefaultUncaughtExceptionHandler方法！defaultUncaughtHandler是Thread类的静态成员变量，所以如果我们将自定义的UncaughtExceptionHandler设置给Thread的话，那么当前进程内的所有线程都能使用这个UncaughtExceptionHandler来处理异常了。
* 一个简易版本的UncaughtExceptionHandler类的子类CrashHandler，CrashHandler的使用方式就是在Application的onCreate方法中设置一下即可

## multidex
> Android中单个dex文件所能够包含的最大方法数是65536，这包含Android Framework、依赖的jar以及应用本身的代码中的所有方法。如果方法数超过了最大值，那么编译会报错DexIndexOverflowException。
有时方法数没有超过最大值，但是安装在低版本手机上时应用异常终止了，报错Optimization failed。这是因为应用在安装的时候，系统会通过dexopt程序来优化dex文件，在优化的过程中dexopt采用一个固定大小的缓冲区来存储应用中所有方法的信息，这个缓冲区就是LinearAlloc。LinearAlloc缓冲区在新版本的Android系统中大小是8MB或者16MB，但是在Android 2.2和2.3中却只有5MB，当待安装的应用的方法数比较多的时候，尽管它还没有达到最大方法数，但是它的存储空间仍然有可能超过5MB，这种情况下dexopt就会报错导致安装失败。

* 在Android 5.0之前使用multidex需要引入android-support-multidex.jar包，从Android 5.0开始，系统默认支持了multidex，它可以从apk中加载多个dex。Multidex方案主要针对AndroidStudio和Gradle编译环境。
	* 在build.gradle文件中添加multiDexEnabled true
	* 添加对multidex的依赖
	* 在代码中添加对multidex的支持，这里有三种方案
		* 在AndroidManifest文件中指定Application为MultiDexApplication
		* 让应用的Application继承自MultiDexApplication
		* 重写Application的attachBaseContext方法，这个方法要先于onCreate方法执行
 * 采用上面的配置之后，如果应用的方法数没有越界，那么Gradle并不会生成多个dex文件；如果方法数越界后，Gradle就会在apk中打包2个或者多个dex文件，具体会打包多少个dex文件要看当前项目的代码规模。在有些情况下，可能需要指定主dex文件中所要包含的类，这个可以通过--main-dex-list选项来实现这个功能。
 * Multidex方案可能带来的问题
 	* 应用启动速度会降低，因为应用启动的时候会加载额外的dex文件，所以要避免生成较大的dex文件；
	* 需要做大量的兼容性测试，因为Dalvik LinearAlloc的bug，可能导致使用multidex的应用无法在Android 4.0以前的手机上运行。
## 动态加载
* 动态加载技术又称插件化技术，将应用插件化可以减轻应用的内存和CPU占用，还可以在不发布新版本的情况下更新某些模块。不同的插件化方案各有特色，但是都需要解决三个基础性问题：资源访问，Activity生命周期管理和插件ClassLoader的管理。
* 宿主和插件：宿主是指普通的apk，插件是经过处理的dex或者apk。在主流的插件化框架中多采用特殊处理的apk作为插件，处理方式往往和编译以及打包环节有关，另外很多插件化框架都需要用到代理Activity的概念，插件Activity的启动大多数是借助一个代理Activity来实现的。
* 资源访问：宿主程序调起未安装的插件apk，插件中凡是R开头的资源都不能访问了，因为宿主程序中并没有插件的资源，通过R来访问插件的资源是行不通的。Activity的资源访问是通过ContextImpl来完成的，它有两个方法getAssets()和getResources()方法是用来加载资源的。具体实现方式是通过反射，调用AssetManager的addAssetPath方法添加插件的路径，然后将插件apk中的资源加载到Resources对象中即可。
* Activity生命周期管理：有两种常见的方式，反射方式和接口方式。反射方式就是通过反射去获取Activity的各个生命周期方法，然后在代理Activity中去调用插件Activity对应的生命周期方法即可。反射方式代码繁琐，性能开销大。接口方式将Activity的生命周期方法提取出来作为一个接口，然后通过代理Activity去调用插件Activity的生命周期方法，这样就完成了插件Activity的生命周期管理。
* 插件ClassLoader的管理：为了更好地对多插件进行支持，需要合理地去管理各个插件的DexClassLoader，这样同一个插件就可以采用同一个ClassLoader去加载类，从而避免了多个ClassLoader加载同一个类时所引起的类型转换错误。

## 反编译和JNI
* 略

## 性能优化
* 布局优化
	* 删除布局中无用的组件和层级，有选择地使用性能较低的ViewGroup；
	* 使用<include>、<merge>、<viewstub>等标签：<include>标签主要用于布局重用，<merge>标签一般和<include>配合使用，它可以减少布局中的层级；<viewstub>标签则提供了按需加载的功能，当需要的时候才会将ViewStub中的布局加载到内存，提供了程序的初始化效率。
	* <include>标签只支持android:layout_开头的属性，android:id属性例外。
	* ViewStub 继承自View，它非常轻量级且宽高都为0，它本身不参与任何的布局和绘制过程。实际开发中，很多布局文件在正常情况下不会显示，例如网络异常时的界面，这个时候就没有必要在整个界面初始化的时候加载进行，通过ViewStub可以做到在需要的时候再加载。
* 绘制优化
	* onDraw中不要创建新的布局对象，因为onDraw会被频繁调用；
	* onDraw方法中不要指定耗时任务，也不能执行成千上万次的循环操作。
* 内存泄露优化
	* 可能导致内存泄露的场景很多，例如静态变量、单例模式、属性动画、AsyncTask、Handler等等
* 响应速度优化和ANR日志分析
	* ANR出现的情况：Activity如果5s内没有响应屏幕触摸事件或者键盘输入事件就会ANR，而BroadcastReceiver如果10s内没有执行完操作也会出现ANR。
	* 当一个进程发生了ANR之后，系统会在/data/anr目录下创建一个文件traces.txt，通过分析这个文件就能定位ANR的原因。
* ListView和Bitmap优化
	* ListView优化：采用ViewHolder并避免在getView方法中执行耗时操作；根据列表的滑动状态来绘制任务的执行频率；可以尝试开启硬件加速来使ListView的滑动更加流畅。
	* Bitmap优化：根据需要对图片进行采样	
* 线程优化
	* 采用线程池
* 其他优化建议
	* 不要过多使用枚举，枚举占用的内存空间要比整型大；
	* 常量请使用static final来修饰；
	* 使用一些Android特有的数据结构，比如SparseArray和Pair等，他们都具有更好的性能；
	* 适当使用软引用和弱引用；
	* 采用内存缓存和磁盘缓存；
	* 尽量采用静态内部类，这样可以避免潜在的由于内部类而导致的内存泄露。
	* MAT是功能强大的内存分析工具，主要有Histograms和Dominator Tree等功能


	