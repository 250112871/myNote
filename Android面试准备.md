Activity :与用户交互的界面

生命周期 oncreate()->onStart()->onResume()

四种状态 running  paused stopped killed

android 进程优先级 
前台  可见  服务 后台 空

android 任务栈

activity 启动模式  standard singleTask  singleTop singleInstance

scheme 跳转协议 

---
Fragment 

第五大组件

加载到Activity 的两种方式 xml  和FragmentManager

FragmentPagerAdapter(把UI分离) 和FragmentStatePager(回收内存) 区别 前者适用于页面较少的情况,后面使用于页面较多的情况 

Fragment生命周期

onAttach ->onCrate->onCreateView->onViewCreate-> Activity-onCreate->onActivityCreate->Activity-onStart->onStart->Activity-onResume->onResume
onPause->Activity-onPause->onStop->Activity-onStop->onDestroyView->onDestory->onDetach->Activity-onDestroy

Fragment 通信

Fragment -Activity 

Activity-Fragment

Fragment-Fragment 

---
Service 可以在后台长期运行操作而没有界面的应用组件 

thread Activity中 Activity销毁后无法对Thread进行控制.

service 启动方式

startService  onBinder

---
Broadcast Receiver
 应用程序之间传递信息的机制 .内容通过Intent 携带要传递的数据

多个进程不同组件之间传递消息.

广播种类:
1 普通广播 Context.sendBroadcast
2 有序广播 Context.sendOrderedBroadcast
3 Local Broadcast 只在自身app内传播

实现广播
动态注册 静态注册

广播内部实现机制
1自定义广播接收者BroadcastReceiver 复写onRecvice()
2通过Binder机制向AMS进行注册
3广播发送者通过Binder机制向Ams发送广播
4AMS查找符合相对应条件的BroadcastReciver,讲广播发送到Broadcastreceiver 相对应的消息列队循环中
5消息循环拿到次广播 回调BroadcastReceiver中的onReceiver()方法.

本地广播

不必要担心数据泄露
其他app无法

WebView 

常见坑 
1 Android 16 及以前的版本 通过Web.addJavascriptInterface 方法  远程攻击者通过反射调用任意java对象的方法
2. webView 在布局文件中的使用 webView 写在其他容器中时 需要先通过 父容器
3. jsbridge 
4. webViewClient.onPageFinished  ->webChromeClient.onProgressChanged
5. 后台耗电	根据业务需求到后台时移除
6. webView硬件加速渲染问题. 页面白块 界面抖动  暂时关闭
 
关于内存泄露
1. 开启独立进程 (用完之后直接干掉这个进程)
2.  动态添加WebView 对传入WebView中的Context 使用弱引用,

Binder 相关

binder 是一种跨进程的通信机制

为什么使用Binder

linux内核进程间的通信机制 管道 ,sockect

1.性能 高效

2.安全 进行了身份校验

Binder通信模型

Aidl

----
为什么Android 必须在主线程更新UI?
Android UI操作并不是线程安全的 ,并且这些操作必须在UI线程执行

Android屏幕刷新机制

1 界面上任何一个View的刷新请求最后都会走到ViewRootImpl中的scheduleTraversals()里来安排一次遍历View树的任务
2 scheduleTraversals() 会先过滤掉同一帧的重复调用,在同一帧内只需要安排一次遍历绘制View书的任务即可,这个任务会在下一个屏幕刷新信号到来时调用performTraversals()遍历View树,遍历过程中讲需要刷新的View进行重回
3 接着scheduleTraversals()会往主线程的消息列队中发送一个同步屏障,拦截这个时刻之后所有的同步消息的执行,但不会拦截异步消息,尽可能的保证当前接收到屏幕刷新信号时可以尽可能第一时间处理遍历绘制View树的工作
4 发完同步屏障后,scheduleTraversals()才会安排一个遍历绘制view树的操作,做法是吧perfromTraversals()封装到Runnabel里面,然后调用Choreographer的postCallBack()方法
5，postCallback() 方法会先将这个 Runnable 任务以当前时间戳放进一个待执行的队列里，然后如果当前是在主线程就会直接调用一个native 层方法，如果不是在主线程，会发一个最高优先级的 message 到主线程，让主线程第一时间调用这个 native 层的方法；
6， native 层的这个方法是用来向底层注册监听下一个屏幕刷新信号，当下一个屏幕刷新信号发出时，底层就会回调 Choreographer 的onVsync() 方法来通知上层 app；
7，onVsync() 方法被回调时，会往主线程的消息队列中发送一个执行 doFrame() 方法的消息，这个消息是异步消息，所以不会被同步屏障拦截住；
8，doFrame() 方法会去取出之前放进待执行队列里的任务来执行，取出来的这个任务实际上是 ViewRootImpl 的 doTraversal() 操作；
9，上述第4步到第8步涉及到的消息都手动设置成了异步消息，所以不会受到同步屏障的拦截；
10，doTraversal() 方法会先移除主线程的同步屏障，然后调用 performTraversals() 开始根据当前状态判断是否需要执行performMeasure() 测量、perfromLayout() 布局、performDraw() 绘制流程，在这几个流程中都会去遍历 View 树来刷新需要更新的View；




Handler

通过发送和处理 Message和Runnable对象来关联对应线程的MessageQueue 

1 可以让对应的Message和Runnable在未来的某个时间点进行对应处理
2 让自己想要处理的好事操作放在子线程,让更新UI的操作放在主线程


原理:
Looper 
MessageQunue

引起内存泄露问题:


AsyncTask

封装了线程池和handler的异步框架


HandlerThread 

封装了Looper的Thread 
避免频繁创建线程,降低资源开销


IntentService 

集成并处理异步请求的一个类,集成了Service 内封封装了 HandlerThread 和 handler 


View 绘制机制

View树的绘制流程

就像一个递归过程在 onMeasure() 方法里面 View会对所有子元素进行测量,测量过程就从父ViewGroup传递到子View的过程,经过子元素的递归,测量好所有子元素的长度后,进行递归,反复之后完成了所有的Viewgroup的测量

measure ->layout->draw

measure()  -> onMeasure()-> setMeasuredDimension()

viewgroup.LayoutParams

MeasureSpec 

UNSPECIFIED
EXACTLY
at most

invalidate()  measure layout 不会调用
requestLayout() measure layout 会被调用 draw方法不一定会被调用

---
事件分发
Activity -> phonewindow->DecorView->ViewGroup->...->View
phonewindow 

decorView

事件分发的机制分发方法

dispatchTouchEvent

onInterceptTouchEvent (Activity 和View  没有这个方法 view不需要拦截,Activity 拦截后整个页面都无法获取)

onTouchEvent

---
ListView 

listView 适配器模式

listView 的recycleBin机制

listView的优化

convertView 重用/viewHolder

三级缓存/监听滑动事件

---
### 打包流程

1. aapt打包项目中的资源文件打包成R.java文件
2. aidl 工具将aidle 文件转换成java接口
3. javaCompiler 将R.java, Application Source Code, Java Interface  打包成.class File文件 字节码
4. dex 变异成.dex files 文件
5. apk builder  打包成apk
6. 通过jarSigneture 对apk进行签名
7. zipAlign

### Jenkins 持续集成构建

### git

### Proguard
 
功能
	1. 压缩 删除无用的文件
	2. 优化  删除无用的字节码文件方法
	3. 混淆 将有意义的名词混淆成
	4. 预检测

工作原理
	EntryPoint 
	
OkHttp 
	使用
		1. 创建一个OkHttpClient
		2. 创建一个request对象,通过内部类的Builder调用生成Request对象
		3. 创建一个Call对象,调用execute/enqueue
		
	源码分析	
	 同步请求

retrofit 
	使用
		1. 创建一个api接口
		2. 创建一个Retrofit示例
		3. 调用api接口	

volley 
	使用
		1. 创建RequestQueue 对象
		2. 创建具体的请求对象
		3. 将请求对象添加到RequestQueue对象里

Butterknife 
	使用
		依托java的注解机制来实现辅助代码生成的框架,编译时在生成代码来查找.
		@BindView(R.id.textView) 绑定View
		@onClick(R.id.textView) 给View增加监听
		@onClick({R.id.textView1,R.id.textView2}) 给多个View增加监听	
		@onItemClick(R.id.listView) 给list 的Item设置点击事件
	原理
		1. 扫描java代码中所有ButterKnife注解
		2. 在ButterKnifeProcessor 生成 <className>$$ViewBinder内部类
		3. 调用binder方法加载生成ViewBinder类

Glide		
	
Anr && OOM
	应用程序无响应的对话框  Activity 5秒  BroadCastReceiver 中 10秒
	
	原因:主线程做了耗时操作   例如:Activity生命周期,Service BroadCastReceiver 
	处理: 
		Activity 的onCreate和OnResume回调中避免耗时操作
		
OOM 
    当前占用的内存加上加上我们申请的内存资源超过的Dalvik虚拟机的最大限制就会抛出 Out Of Memery		
	
	处理:
	关于bitmap
	1. 图片展示 滑动的控件,当停止滑动的时候在加载图片
	2. 及时释放内存 java 部分和C部分
	3. 图片压缩
	4. inBitmap属性
	5. 捕获异常Error异常
	
	listView: convertView   / lru
	避免在onDraw 方法里执行对象的创建
	谨慎使用多进程 


内存抖动

内存泄露

Bitmap  
	1. recycle 
	2. LRU 内部用linkedHasMap<>内部 
	3. 计算 inSampleSize 缩放比例
	4. 缩略图
	5. 三级缓存 本地 内存 网络

UI 卡顿
	UI卡顿原理 60fps-> 16ms 一次刷新信号 大量GC 层高过深，动画过 
	1. 在UI线程中做经微的耗时操作，导致UI卡顿
	2. 布局layout过于复杂，无法在16ms内完成渲染
	3. 同一时间动画执行次数过多，导致cpu或GPU负载过过重
	4. View过度绘制，导致某些像素在同一帧时间内被绘制多次，从而使cpu或gpu负载过重
	5. view 频繁的触发 measure layout  导致measure layout 累计耗时过多及整个View频繁的重新渲染
	6. 内在频繁触发GC，导致暂时阻塞渲染操作
	7. 冗余资源及逻辑等导致加载和执行缓慢
	8. ANR
	
	1. 布局优化  inclue  过于复杂可以考虑算定义View
	2. 列表及Adapter 优化滑动   
	3. 背景和图片等内在分配优化
	4. 避免ANR  

内在泄漏
	内在泄漏基础知识
	1. java内存的分配策略
		静态存储区 
		栈区  一般在方法中定义的基本类型变量 和引用型变量 
		堆区  new 出来的对象  有垃圾回收机制进行回收
	java是如何管理内存的
		通过new 分配内存，通过gc回收
	
	java中的内存泄漏
		无用的对象持续占有内存或者无用对象的内存得不到及时释放，从而造成的内存空间的浪费成为内存泄漏
	
	Android 内存泄漏
		1. 单利模式 context context.getApplicationContext()
		2. 匿名内部类  内部如果有个静态实例  那么它将同应用一样长  内部类用静态
		3. Handler  引用Activity 使用 weakReference<>
		4. 避免使用static变量 生命周期管理起来
		5. 资源未关闭造成的内存泄漏
		6. AsyncTask
内存管理
	内存管理机制概述
		分配机制  
		回收机制
	Android 内存管理机制
		 1. 分配机制 分配一个小额的内存 随着apk的运行在分配额外的内存 有上限  让更多的进程存活在内存中，当用户在启动apk的时候，只需要回复已有的进程，可以提高用户体验 
		 2. 回收机制 前台进程 可见进程 后台进程 服务进程 空进程

	内存管理机制的特点		
		