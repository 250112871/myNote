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
		1. 更少的占用内存
		2. 在合适的时候,合理的释放系统资源
		3. 在系统内存紧张的情况下,能释放掉大部分不重要资源,为Android系统提供可用的内存
		4. 能够在合理的在特殊生命周期中,保存或者还原重要数据,以至于系统能够正确的重新恢复改
		
内存优化
	1. 当Service完成任务后,尽量停止它.
	2. 在UI不可见的时候,释放掉一些只有UI使用的资源
	3. 在系统内存紧张的时候,尽可能多的释放掉一些非重要的资源
	4. 避免滥用Bitmap 
	5. 使用针对内存优化过得数据容器 
	6. 避免使用依赖注入的框架
	7. 使用zip对齐的apk
	8. 使用对进程

内存溢出vs内存泄漏
	1.内存溢出
	2. 内存泄漏
	
冷启动
	什么是冷启动
		在应用启动前,系统中没有该应用的任何进程信息.
	
	2热启动
		用户使用返回键退出应用,然后马上又重新启动应用.
	
冷启动时间的计算
	应用启动开始计算,到完成视图的第一次绘制(即Activity内容对用户可见)为止
	
	冷启动流程
		zygote进程中fork创建出一个新的进程
		创建和初始化Application类,创建MainActivity类
		inflate布局,当onCreate/onStart/onResume方法走完 
		contentView的onmeasure/layout/draw 显示在界面上
		
		Application 的构造其方法->attachBaseContext()-> onCreate()->Activity的构造方法->onCreate()->配置主题中的背景等属性->onStart()->onResume()->测量布局绘制显示在界面
		
冷启动优化
	1. 减少onCreate()方法的工作量  初始化操作可以放到 Application中
	2. 不要让Application参与业务的操作
	3. 不要再Application进行耗时操作
	4. 不要以静态变量的方式在Application中保存数据
	5. 布局/mainThread
	
其他优化
	android中不用静态变量存储数据. 系统杀掉后悔重新启动此时 静态存储数据可能会不安全 
	
	sharepreference 问题 不要进行跨进程的操作,每个进程都会维护一份自己的Sharepreference的副本,结束时同不
	存储Sharepreference的文件过大的问题, 	
		1. 阻塞主线程.
		2. 解析比较大的对象时,会产生临时对象,频繁的GC 大量的Gc造成内存抖动
		
	内存对象序列化
		1.Serializeble  本地存储时用.会产生大量临时变量
		2. Parcelable  跨进程传递时更方便
		
	UI线程做繁重的操作
	
MVC 
	M:业务逻辑处理
	V:处理数据展示部分
	C:control Activity处理用户交互问题 
	
	便于UI显示 和业务逻辑的分层  时项目有了很好的可扩展和维护行
	
MVP
	M:业务逻辑和实体模型
	V:对应Activity负责View的绘制及用户交互
	P:完成View和Model间的交互
	
	MVC 和 MVP最大的区别是 V,M  是否可以相互操作
	
MVVM	
	M:业务逻辑和实体模型
	V:对应Activity负责View的绘制及用户交互 没有业务逻辑 
	ViewModel 业务逻辑在这处理  View与Model间的交互
	
	数据绑定
	
### Android插件化
	1. 动态加载apk DexClassLoader apk文件中的字节码  PathClassLoader 加载目录中的字节码
	2. 资源加载 Assetmanager
	3. 代码加载 生命周期反射调用

### 热更新
	热更新流程
	1. 线上检测到严重的crash
	2. 拉出bugfix分支并在分支上修复问题
	3. jenkins构建和补丁生成
	4. app通过推送或者主动拉取补丁文件
	5. 将bugfix代码合到master上
	
主流更新框架介绍
	Dexposed alibaba aop思想 hook形式  基动态类加载技术,运行时app可以加载一小段经过编译的代码

	AndFix 思想同Dexposed相似 专注于热修复,没有其他功能
	
	Nuwa 基于Android分包技术

### 热更新原理
		
	1. PathClassLoader
	2. DexClassLoader
	
原理
	dexElements 数组会在BaseClassloader中加载好,将需要替换的类打包成dex包 放到dexElements数组最前面 BaseClassload加载到后就不会加载后面的Dex包中的类实现了替换功能.

进程保活 

	1. android进程的优先级 
		前台进程 可见进程 后台进程(LRU算法) 服务进程 空进程(缓存用)
	2. android进程的回收策略
	3. 进程保活方案

### 进程回收策略
	1. Low memory killer: 定时进行检查 通过一些比较复杂的评分机制,对进行打分,然后将分数高的进程判定为bad进程,杀死并释放内存.
	2. OOM_ODJ:判断进程优先级

### 保活方案
	1. 利用系统广播拉活  开机广播,
	2. 利用系统Service 机制拉活 第一次杀死5秒重启 第二次10秒 第三次20秒 
	3. 利用native进程拉活,5.0之后就失效了 定时监控
	4. 利用JobScheduler机制拉活
	5. 利用账号同步机制拉活. 	
	
Java 

基础知识
	1. ip地址和端口号
	2. tcp/udp 协议
	
Socket
	客户端 
	服务端

类加载器 
	启动类加载器
	扩展类加载器
	应用程序加载器
	自定义加载器
		
类加载的过程
	加载 验证 准备 解析 初始化 使用 卸载
	
	加载
	1. 通过一个雷的全限定名来获取其定义的二进制字节流
	2. 将这个字节流所代表的的静态存储结构转化为方法区的运行时数据结构
	3. 在java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口
	
	验证
	1. 为了确保class文件中的字节流包含的信息符合当前虚拟机的要求
	2. 文件格式的验证，元数据的验证，字节码验证和符号引用验证
	
	准备
	1. 基本数据类型  不显示赋值则 默认赋值
	2. 同时被static和final修饰的常量  声明时就要赋值
	3. 对引用数据类型 reference 
	4. 在数组初始化 根据数据类型 附初始值
	
	解析
	1. 类或接口的解析 所转换的直接引用是 普通类 类型或者数组类型
	2. 字段解析 是否包含名称 没有按照继承关系 从上向下依次寻找
	3. 类方法解析  先搜索父类 在搜索接口
	4. 接口方法解析
	
	初始化 

java程序运行时的内存分配策略
	1.静态处处区 主要存放静态数据，全局static数据和常量
	2.栈区：方法体内的局部变量都在栈上的创建
	3.堆区： 程序直接new出来的内存

从内存分配角度	
	在方法体内定义的基本类型和对象的引用变量都爱方法栈内存中分配
	
	堆内存用来存放所有有new 创建的对象（包括对象其中的所有成员变量）和数组。在堆中分配的内存，将有java垃圾回收器来自动管理

java 内存回收机制
	引用原则，

java 内存泄漏引起的原因
	长生命周期对象持有短生命周期的对象引用 容易引起内存泄漏
	
	堆：运行时，垃圾回收，动态分配内存，存取速度（比栈慢）
	
	栈：存取速度 生命周期
	
java 反射
	编译时：将java代码编译成 .class文件的过程 不涉及内存
	运行时：就是Java虚拟机执行.class文件的过程 涉及内存调用
	
编译时类型和运行时类型
	编译时类型： 编译时类型由声明该变量时使用的类型决定
	运行时类型：运行时类型有实际赋给该变量的对象决定	

反射	在运行状态中，对于任意一个类，都能知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态的获取信息及调用方法，叫反射机制。

class类获取方式 
	1. Person person = new Person();
	   Class<? extends Person> personClass = person.getClass();   //可以通过class 获取报名，类名
	2. Class<?> personClass = Class.forName("Person");  
	3. class<? extends Person > personClass = Person.class();

Android 中反射的运用
	1.通过原始的Java反射机制的方式调用资源
		
	2. Activity的启动过程中Activity的对象创建
	
IO
	阻塞IO
	BIO
	NIO 自己实现下 

多线程
	启动方式
		复写Thread类 
		实现Runnable 接口方法
		
		线程间通信
		1. Synchronized 利用java虚拟机机制 悲观锁 独占锁
		2. volatile 
		3. lock		java代码 乐观锁
	
sleep/wait
	主要区别 锁
wait / notify	
	锁
	
多线程
	1. 降低资源消耗
	2. 提高响应速度
	3. 线程池提高线程的可管理性

工作流程：
	1. 判断线程池基本线程池是否已满。
	2. 判断县城吃工作列队是否已满
	3. 最后判断整个县城池是否已满

异常处理原理
	java虚拟机用方法栈来跟中每个线程中一些列的方法调用过程
	如果执行方法的过程中抛出异常 则java虚拟机必须找到能捕获该异常的catch代码块
	当java虚拟机追溯到调用栈的底部的方法是，如果仍然没有找到处理该异常的代码块 则打印异常信息，如果不是主线程终止线程。

异常流程
	1. finally 语句不被执行情况 try中执行 System.exit(0)  
	2. try 中 执行return finally 会在 try 返回前执行
	3. final 语句中不能改变try中的结果
	4. 建议finally 代码块中不要使用 return语句 数据不安全

可检查异常 非检查异常
	经编译验证对于声明抛出的异常任何方法 编译器强制要求处理的声明规则
	不准寻声明规则，不去检查是否处理了异常  RuntimeException Error 
	
throw 和 throws 

final finally finalize 区别
	finalize Ojbect的方法 当虚拟机回收对象时会调用对象的finalize方法
	
注解

Annotation（注解）：java提供了一种元程序中的元素关联任何信息和任何元素局的途径和方法  

基本的规则：不能影响程序代码的执行，

metadata：描述数据的数据
	以标签的形式存在代码中
	描叙的信息是类型安全的
	元数据需要编译器之外的工具额外处理用来生成其他的程序部件
	元数据可以存在于java远吗级别，也可以存在于编译后的class文件内部。

注解分类
	1. 系统内置注解  @Overider @Deprecated  @SuppressWarnings 
	2. 元注解 
		@Target 类型(ElementType.Type, ElementType.FIELD,ElementType.CONSTRUCT) 
		@Retention 保留时长 （RetentionPolicy.RUNTIME ,RetentionPolicy.SOURCE,RetentionPolicy.CLASS）
		@Documented 需要标注api
		@Inherited 可以被继承的
		
设计模式

单利模式
	对象创建模式，它用于生产一个对象的具体事例，它可以确保系统只创建一个对象
	
	节省系统开销
	由于new 的次数变少减少GC
	
	1. 饿汉式
	2. 懒汉式 优化双非空判断
	3. 静态内部类  
	4. 枚举
	
Builder
	较为复杂的创建型模式，客户端包含多个组成部件的复杂对象的创建分离
	
使用场景	
	当构造一个对象需要很多参数的时候，并且参数的个数或者类型不固定的时候。
	
HTTP协议
	协议：计算机通讯网络中，两台计算机之间进行通信锁必须共同遵守的规定或规则
	HTTP协议:超文本传输协议的一种通信协议，它允许将超文本标记语言文档从web服务器传送到客户端的浏览器
	URI：uniform resouur identifier,统一资源标识符，用来唯一的标识一个资源
	三个部分组成 
	1. 访问资源的命名机制
	2. 存放资源的主机名
	3. 资源自身的名称，有路径标识，强调的资源
	
	URL： uniform resource locator 统一资源定位器，他是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源
	1. 协议
	2. 存有该资源的主机IP地址
	3. 主机资源的具体地址。
	
	HTTP协议的特点
	1. 简单快速
	2. 无连接
	3. 无状态
	
	请求头信息
		GET  请求方法
	Host 告诉我们请求主机的地址和端口号 
	User-Agent 客户端的操作系统和浏览器的名称
	Intervention 
	Accept： 浏览器端可以接受的媒体类型
	Referer：服务器从哪个地址链接过来的
	Accept-Encoding 声明编码方法 通常是否可以压缩
	Accept-Language  声明语言
	if-None-Match  判断内容是否改变的 
	if-Modified-Since 数据修改的时间
	
	返回信息
		HTTP、1.1 304 NotModified     返回状态吗  304内容没有改变
	Server 服务信息
	Date  日期   生成消息的具体时间
	Last-Modified：  上面的信息最后修改的时间 和上面  if-Modified-Since-since 对应
	
	ETag 同上面if-None-Match-Match 对应
	Expires 缓存的有效时间
	cache-Control 遵循的缓存机制
	proxy-Connection keep-alive

	http1.1 /http 1.0 具体区别
	1. 缓存处理                缓存方式 1.1 引入了更多的缓存策略
	2. 带宽优化及网络连接的使用   1.11 可以请求某个部分内容
	3. host头处理 针对一台物理主机有多个虚拟主机情况
	4. 长连接
	
	存在问题
	1. 数据传输时每次都需要建立连接，增加了大量的延迟时间		1.1 处理
	2. 传输数据的内容都是明文 客户端和服务端无法验证对方的身份  https 处理
	3. header里携带的内容过大，在一定程度上增加了传输的成本
	4. 虽然 1.1 支持了Keep-alive 来弥补多次创建连接产生的延迟，但keep-alive使用同样会给服务端带来大量的性能压力。
	
	get/post 方法的区别
	get 获取资源
	post 更新资源
	
	1 提交数据 get：在URL之后用？分割。 post：放在body中
	2 提交数据大小是否有限制 get 有限制 
	3 取得变量的值 Request.QueryString vs Request.Form
	4 安全问题
	
	cookie 和 session的区别
	
	Cookie技术是 客户端的解决方案， Cookie是由服务端发给客户端的特殊信息，这些信息以文本文件的方式存放到客户端，然后客户端每次向服务端发送请求的时候都会带上这些特殊的信息 
	
	session 将用户信息保存在服务端
	
	区别
	1. 存放位置不同   客户端 服务端
	2. 存取的方式不同 字符串  任何类型
	3. 安全性的不同   浏览器中可见  服务端
	4. 有效期上不同   设置有效期  发送session 值-1      
	5. 对服务造成的压力不同
	
HTTPS 
	https不是一个单独的协议，而是对工作在-加密连接（SSL/TLS）上的常规HTTP协议。通过在TCP和HTTP之间加入TLS（Transport Layer Security）来加密
	
SSL/TLS 协议	
	
	1. 通信慢
	2. ssl必须进行加密处理

TLS/SSL握手
1. 密码学原理
	对称加密 加密 解密 秘钥是一样的
	不对称加密 
		私有
2. 数字证书 标识身份的一串数字

3 SSL与TLS握手的原理 
	
ssl TLS  握手过程
	
1. 客户端发起请求  将 支持的协议版本 支持的加密及压缩算法  产生一个随机数 扩展内容 发送给服务端
2. 服务端回应， 确定的协议及加密算法 服务端证书 服务端随机数 扩展内容
3. 客户端验证证书  随机数（使用证书中的公钥加密） 编码改变通知，握手结束通知  
4. 生成秘钥  编码改变通知 握手偶结束
5. 客户端发送数据 	

TCP/IP 网络模型
	1. 物理层/实体层 物理手段链接电脑的手段
	2. 链接层  将电子信号组织成数据结构
		1. 链接层协议  以太网协议规定了电信号的分组方式 侦 head data
		2. mac 网卡都有mac地址
		3. 广播  同一个子网络相互传递
	3. 网络层 	主机到主机之间的通信	使得我们能够区分不同的计算机是否属于同一个子网络 ，能够区分哪些Mac地址属于同一个子网络 
		ip协议 规定网络地址的协议
		ip 数据包 根据ip协议发送的数据包
		
		arp协议
			TODO：需要一个机制 能够从IP地址找到MAC地址 ，同时知道两个地址。
		核心原理：
			ARP协议也是发送一个数据包，它所在子网络的每一台主机都会收到这个数据包，解析这个数据包里的ip地址 同自身的ip地址比较相同做出回复 并上报自己的mac地址，否则丢弃这个包。
		 
	4. 传输层
		建立端口到端口的通信 
	5. 应用层
		规定应用程序的数据格式。
	
DNS 将域名转换成ip地址的过程
    1. 本地hosts 
	2. 本地DNS解析缓存
	3. TCP/IP参数中设置的首选DNS服务器，具有权威性
	4. 如果要查询的域名 不在本地DNS服务器区域解析，但该DNS服务器已缓存了此网址的映射关系，则调用这个ip地址映射，完成域名解析。
	5. 本地DNS就把请求发送至13台跟DNS，改DNS服务受到请求后会判断这个域名是谁来授权管理，并返回一个负责该顶级域名服务器的一个IP，本地DNS服务器接收到IP信息后，将会里兰溪负责这个域名的这台服务器。
	6. 如果用的是转发模式，此DNS服务器就会把请求转发至上一级DNS服务器，由上一级服务器进行解析。
	
密钥：
	对称
		DES，3DES，AES，RC5，RC6
	不对称
		RSA加密

数字签名
	用于验证传输的内容是不是真实服务器发送的数据，发送的数据有没有被篡改过。 是非对称加密的一种场景，不过他是反过来用私密进行加密，通过与之匹配的公钥来解密。
	
	数字签名过程
	
	数字证书
	
