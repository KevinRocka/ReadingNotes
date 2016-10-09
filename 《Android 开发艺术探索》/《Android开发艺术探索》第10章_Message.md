#Chapter 10.Message

Time : 2016.10.9

Author : Rocka 

##  前言
* Handler的作用轻松的将一个任务轻松的切换到Handler所在线程中执行，更新UI只是其中的一个

* Android 消息机制主要是指Handler的运行机制，Handler运行需要MessageQueue和Looper的支撑。MessageQueue名字叫消息队列，真正的数据结构是单链表来存储数据。Looper会以无限勋魂的形式去查是否有新消息，有就处理消息，否则一直等待着。

* Looper中有个ThreadLocal，它并不是线程，作用是每个线程中存储数据，在不同线程中互不干扰的提供数据。通过ThreadLocal可以轻松拿到每个线程的Looper。线程默认没有Looper的。如果需要使用Handler必须为线程创建Looper


## Android消息机制概述

* 子线程访问UI跑出异常，这个操作检测是在ViewRootImpl中的checkThread()进行验证的，就是经常遇到的Only the original thread that created a view hierarchy can touch this views

* Handler的创建会采用当前线程的Looper来构建内部的消息循环系统，没有Looper会报错，can't create handler inside thread that has not called Looper.prepare()

* Handler可以用post方法将一个Runnable投递到Handler内部的Looper中去处理，也可以用send方法发送一个消息投递到消息队列中，这个消息同样会在Looper中去处理，其实post最终还是调用了send方法。注意：Looper是运行在创建Handler所在的线程中的，这样一来Handler的业务逻辑就被切换到创建Handler的线程中去了。
 

## Android消息机制分析

* ThreadLocal
	* 是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，对于其他线程来说则无法获取到数据。一般来说，当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，可以考虑使用ThreadLocal。 对于Handler来说，它需要获取当前线程的Looper，而Looper的作用域就是线程并且不同线程具有不同的Looper，这个时候通过ThreadLocal就可以实现Looper在线程中的存取了。
	* 不同线程访问同一个ThreadLocal的get方法，ThreadLocal内部会从各自的线程中取出一个数组，然后再从数组中根据当前的ThreadLocal索引去在查找对应的value值。
	* ThreadLocal是一个泛型类，ThreadLocal数据的存储规则：ThreadLocal的值在table数组中的存储位置总是ThreadLocal的索引+1的位置。ThreadLocal的set和get方法看出，所操作的对象都是当前线程的localValues对象的table数组，不同线程访问同一个Threadlocal的set和get方法，对ThreadLocal所做的读/写操作仅限于各自线程内部

* MessageQueue
	* 包含两个操作，插入（enqueueMessage）,读取（next）;读取操作是从消息队列中取出一条消息并从消息队列移除，MessageQueue实际上是一个单链表的数据结构来维护消息队列
	* next方法是一个无线循环的方法，如果消息队列没有消息，next方法会一直阻塞在这里。当有新消息来的时候，next方法会返回这条消息并且从单链表中删除。

* Looper
	* Looper扮演者消息循环的角色，不停地从MessageQueue中查看是否有新消息，有新消息立即处理，没有就阻塞在那里
	* 为一个线程创建Looper的方法
	
	```
	new Thread("test"){
	    @Override
	    public void run() {
	        Looper.prepare();//创建looper
	        Handler handler = new Handler();//可以创建handler了
	        Looper.loop();//开始looper循环
	    }
	}.start();
	``` 
	* Looper 的quit(直接退出Looper) 和 quitSafely(设定一个标记，消息处理完才退出)，Looper退出之后，通过Handler发送的消息就会失败，这个时候Handler的send方法会返回false。
	* 在子线程中，如果手动为其创建了Looper，那么在所有的事情完成以后应该调用quit方法来终止消息循环，否则这个子线程就会一直处于等待的状态，而如果退出Looper以后，这个线程就会立刻终止，因此建议不需要的时候终止Looper。
	* Looper的loop方法会调用MessageQueue的next方法来获取新消息，而next是一个阻塞操作，当没有消息时，next方法会一直阻塞着在那里，这也导致了loop方法一直阻塞在那里。如果MessageQueue的next方法返回了新消息，Looper就会处理这条消息：msg.target.dispatchMessage(msg)，其中的msg.target就是发送这条消息的Handler对象。这样Handler发送的消息最终又给它的dispatchMessage方法来处理了。但是这里不同的是，Handler的dispatchMessage方法是在创建Handler时所使用Looper中执行，这样就成功的将代码逻辑切换到指定线程中
* Handler 
	* 消息处理
	
	```
	public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);//当message是runnable的情况，也就是Handler的post方法传递的参数，这种情况下直接执行runnable的run方法
        } else {
            if (mCallback != null) {//如果创建Handler的时候是给Handler设置了Callback接口的实现，那么此时调用该实现的handleMessage方法
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);//如果是派生Handler的子类，就要重写handleMessage方法，那么此时就是调用子类实现的handleMessage方法
        }
    }
	``` 
	* 两种方式实现Handler：
		* Handler handler = new Handler(callback)，callback中处理消息
	  	* 派生一个Handler的子类并重写handleMessage

	
## 主线程消息循环

* Android的主线程就是ActivityThread，主线程的入口方法就是main，其中调用了Looper.prepareMainLooper()来创建主线程的Looper以及MessageQueue，并通过Looper.loop()方法来开启主线程的消息循环。主线程内有一个Handler，即ActivityThread.H，它定义了一组消息类型，主要包含了四大组件的启动和停止等过程

*  ActivityThread通过ApplicationThread和AMS进行进程间通信，AMS以进程间通信的方法完成ActivityThread的请求后会回调ApplicationThread中的Binder方法，然后ApplicationThread会向H发送消息，H收到消息后会将ApplicationThread中的逻辑切换到ActivityThread中去执行，即切换到主线程中去执行，这个过程就是主线程的消息循环模型。
