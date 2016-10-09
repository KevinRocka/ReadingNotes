#Chapter 3.View_Event

Time : 2016.8.5 - 2016.8.10

Author : Rocka 


## View基础

- 位置坐标
 * View在平移过程中，top和left是原始左上角的位置信息，其值不会改变，发送改变的是x，y(View左上角的坐标)，translationX和translationY(View左上角相对于父容器便宜量)
- MotionEvent
 * 点击View产生x，y，而getX/getY返回相对于当前View左上角x和y的坐标，getRawX/getRawY返回的是相对屏幕设计左上角x和y坐标
- TouchSlop
 * 系统认定滑动最小常量：ViewConfiguration.get(getContext()).getScaledTouchSlop().源码中找到有8dp
- VelocityTracker(速度追踪)

	```
	// 在View的OnTouchEvent()方法中追踪手指滑动速度
	VelocityTracker tracker = VelocityTracker.obtain();
	tracker.addMovement(event);
	tracker.computeCurrentVelocity(1000);//滑动时间间隔
	int xVelocity = (int)tracker.getXVelocity();
	int yVelocity = (int)tracker.getYVelocity();
	
	//最后在不需它的时候clear重置回收内存
	tracker.clear();
	tracker.recycle();
	```
	
- GestureDetector(手势检测)
  
  ```
  //1.
  GestureDetector mGestureDetector = new GestureDetector(this);
  mGestureDetector.setIsLongPressEnabled(false);//解决长按屏幕后无法拖动的现象
  //2.实现OnGestureListener接口
  //3.接管目标View的onTouchEvent方法
  boolean consume =  mGestureDetector.onTouchEvent(event);
  return consume;
  ```
 * onFling(快速华东和)
 * 建议监听滑动相关自己在onTouchEvent中实现，监听双击行为可以使用GesureDetector
- Scroller(弹性滑动)
 * scrollTo/scollBy瞬间完成，可以通过scroller.startScroll和computeScroll配合使用完成弹性滑动


## View滑动
 * scrollBy基于当前位置的相对滑动，scrollTo是基于参数的绝对滑动。
 * scrollBy和scrollTo只能改变内容的位置不能改变View在布局中的位置 


## View弹性滑动
  * Scroller工作原理：scroller本身不能让View进行滑动，需要View的computeScroll方法才能完成弹性滑动的效果，不断让View重绘，每次滑动距起始时间有一个时间间隔，通过间隔可以得出View当前的滑动位置，然后就可以通过srollTo完成view的滑动，这样不断重绘，不断滑动。

## View事件分发
### 事件分发
  * 核心伪代码：
  	
  	```
  	public void dispatchTouchEvent(MotionEvent ev){
  		boolean consume = false;
  		if(onIntercepterTouchEvent(ev)){
  			consume = onTouchEvent(ev);
  		} else{
  			consume = child.dispatchTouchEvent(ev);
  		}
  		
  		return consume;
  	}
  
  	``` 
  	
  	解析：一个ViewGroup，点击事件产生后，首先传递给它，它的dispatchTouchEvent()会被首先调用，如果它的onIntercepterTouchEvent()返回true，表示事件交给ViewGroup处理，它的onTouchEvent()方法会被调用;如果返回false就表示不拦截当前事件，事件传递给子元素，子元素的dispatchTouchEvent()会被调用
  	
 * 给View设置OnTouchListener，其优先级比onTouchEvent要高，而onClickListener优先级最低，事件传递尾端
 * 如果一个view的onTouchEvent()返回false，父容器onTouchEvent()会被调用，如果所有元素都不处理事件，就交给activity的onTouchEvent()
 * 某个View一旦开始处理某个事件，这个事件的序列都只有它来处理，并且它的onInterceptTouchEvent不再被调用
 * View的onTouchEvent()默认消耗此事件，返回true,除非同时设置clickable和longclickable同时为false。View的enable属性不影响onTouchEvent返回值，哪怕一个view是disable的状态，只要clickable和longclickable有一个为true，onTouchEvent就会返回true消耗此事件
 * 事件传递是由外向内，即事件总是先传递给父元素，然后由父元素分发给子view

### 事件分发源码解析
 * Activity对点击事件分发过程
 	* 首先事件开始交给Activity所属的Window进行分发，返回true，循环结束，false没人处理，所有View的onTouchEvent都返回false，Activity的onTouchEvent就会被调用。
 	* Window的实现类PhoneWindow通过过superDispatchTouchEvent将事件直接传递给了DecorView(一般就是当前页面的底层容器，即setContentView所设置的父容器，可以通过getWindow().getDecorView()获得)
 	* 由于DecorView继承自FrameLayout且是父View，所以最终事件会传递给View，顶级View一般是ViewGroup。得到Activity所设置的View方式：
 	
 		```
((ViewGroup)getWindow().getDecorView().findViewById(android.R.id.content)).getChildAt(0);
	 
	 	```
 * 顶级View对事件分发过程
 	* 事件到达顶级View(一般是ViewGroup)，会调用ViewGroup的dispatchTouchEvent方法，如果顶级ViewGroup拦截事件，即onInterceptTouchEvent返回true，事件由ViewGroup进行处理。这时如果monTouchListener被设置，onTouch会被调用，否则onTouchEvent会被调用，都提供的话onTouch会屏蔽掉onTouchEvent。onTouchEvent中，设置了mOnClickListener，则onClikc会被调用。如果顶级ViewGroup不拦截事件，则事件会传递给它所在点击事件的子View，子View的dispatchTouchEvent会被调用。
 	* 事件传递过程总是先传递给父元素，然后再由父元素分发给子view，通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外，即当面对ACTION_DOWN事件时，ViewGroup总是会调用自己的onInterceptTouchEvent方法来询问自己是否要拦截事件。ViewGroup的dispatchTouchEvent方法中有一个标志位FLAG_DISALLOW_INTERCEPT，这个标志位就是通过子view调用requestDisallowInterceptTouchEvent方法来设置的，一旦设置为true，那么ViewGroup不会拦截该事件,子View调用request-DisallowInterceptTouchEvent方法并不能影响ViewGroup对ACTION_DOWN事件的处理
	* 便利ViewGroup所有子元素，然后判断子元素是否能够接收到点击事件，由两点判断，子元素是否在播放动画和点击事件的坐E标是否落在子元素区域内。满足两个条件，事件传递给它处理，如果子元素dispatchTouchEvent返回true，暂时不考虑子元素内部怎么分发的，子元素dispatchTouchEvent返回false，就把事件发给下一个子元素。
 * View对点击事件处理过程
   * onTouchListener的优先级高于onTouchEvent，View处于不可用状态下点击事件照样会被消耗
   * 当ACTION_UP事件发生时，会触发performClick方法，如果设置了OnClickListener，会回调onCLick
   * View的LONG_CLICKABLE属性默认为false，而CLICKABLE属性是否为false和view有关，可点击的View(Button)其CLICKABLE为true，不可点击的View(TextView)其CLICKABLE为false。setOnClickListener和setOnLongClickListener分别改变其默认值为true

## View滑动冲突
* 滑动冲突的场景
 	* 外部滑动方向和内部滑动方向不一致
 	* 外部滑动方向和内部滑动方向一致
 	* 上面情况的嵌套
* 处理规则
	* 可以根据滑动距离和水平方向形成的夹角；或者根绝水平和竖直方向滑动的距离差；或者两个方向上的速度差等
* 处理方法
	*	外部拦截法(点击事件都先经过父容器的拦截处理，如果父容器需要此事件就拦截，如果不需要就不拦截。该方法需要重写父容器的onInterceptTouchEvent方法，在内部做相应的拦截即可，其他均不需要做修改。)
	
	```
	public boolean onInterceptTouchEvent(MotionEvent event) {
    	boolean intercepted = false;
    	int x = (int) event.getX();
    	int y = (int) event.getY();

	    switch (event.getAction()) {
	    case MotionEvent.ACTION_DOWN: {
	        intercepted = false;
	        break;
	    }
	    case MotionEvent.ACTION_MOVE: {
	        int deltaX = x - mLastXIntercept;
	        int deltaY = y - mLastYIntercept;
	        if (父容器需要拦截当前点击事件的条件，例如：Math.abs(deltaX) > Math.abs(deltaY)) {
	            intercepted = true;
	        } else {
	            intercepted = false;
	        }
	        break;
	    }
	    case MotionEvent.ACTION_UP: {
	        intercepted = false;
	        break;
	    }
	    default:
	        break;
	    }
	
	    mLastXIntercept = x;
	    mLastYIntercept = y;
	
	    return intercepted;
	}
	``` 
	* 内部拦截法（父容器不拦截任何事件，所有的事件都传递给子元素，如果子元素需要此事件就直接消耗掉，否则就交给父容器来处理。这种方法和Android中的事件分发机制不一致，需要配合requestDisallowInterceptTouchEvent方法才能正常工作，重写子元素的dispatchTouchEvent方法）
	
	```
	//子元素
	public boolean dispatchTouchEvent(MotionEvent event) {
	    int x = (int) event.getX();
	    int y = (int) event.getY();
	
	    switch (event.getAction()) {
	    case MotionEvent.ACTION_DOWN: {]
	        getParent().requestDisallowInterceptTouchEvent(true);
	        break;
	    }
	    case MotionEvent.ACTION_MOVE: {
	        int deltaX = x - mLastX;
	        int deltaY = y - mLastY;
	        if (当前view需要拦截当前点击事件的条件，例如：Math.abs(deltaX) > Math.abs(deltaY)) {
	            getParent().requestDisallowInterceptTouchEvent(false);
	        }
	        break;
	    }
	    case MotionEvent.ACTION_UP: {
	        break;
	    }
	    default:
	        break;
	    }
	
	    mLastX = x;
	    mLastY = y;
	    return super.dispatchTouchEvent(event);
	}
	```
	
	```
	/**
	 * 父元素修改，为什么父容器不能拦截ACTION_DOWN事件？因为ACTION_DOWN不受
	 * FLAG_DISALLOW_INTERCEPT这个标记位控制，所以一旦父容器拦截DOWN事件，
	 * 所有事件无法传递到子元素中 
	 */
	public boolean onInterceptTouchEvent(MotionEvent event){
		int action = event.getAction();
		if(action == MotionEvent.ACTION_DOWN){
			return false;
		}else{
			return true;
		}
	}
	```