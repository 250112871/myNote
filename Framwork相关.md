谈谈你对zygote的理解 ？ （what how why）
zygote的作用
	1. 创建SystemServer    常用类 JNI函数 主题资源 共享库
	2. 孵化应用进程

zygote的启动流程
	启动三段式
		进程启动->准备工作->Loop循环
	怎么启动的	
	Init进程加载启动配置文件（init.rc） ，其中配置了哪些需要启动的服务 zygote  SystemServer
	
	启动进程 
	fork  会返回两次   父进程pid == 子进程pid  子进程pid == 0； 根据pid ==0 判断是子进程或者父进程
	
	两种方式：
	fork+handle    fork+execve 加载的二进制文件会替代继承的父进程资源
	
信号处理- SIGCHLD
	父进程 fork 子进程 子进程挂掉时 会收到SIGCHLD 信号。
	
Zygote 启动之后做了什么
	Zygote的Native世界 []{E:\myNote\img\FrameWorke\Native切换到Java代码.png}
		启动虚拟机->注册Android的JNI函数->进入Java世界  
		
	Zygote的java世界 []{E:\myNote\img\FrameWorke\Zygote的java世界.png}
		Preload Resources(预加载资源) 类主题相关资源 共享库
		启动SystemServer 单独进程
		启动Loop循环 等待  socket消息  处理 []{E:\myNote\img\FrameWorke\Loop处理过程.png}


为什么 要用zygote 孵化进程 不是SystemServe
		我们知道，应用在启动的时候需要做很多准备工作，包括启动虚拟机，加载各类系统资源等等，这些都是非常耗时的，如果能在zygote里就给这些必要的初始化工作做好，子进程在fork的时候就能直接共享，那么这样的话效率就会非常高。这个就是zygote存在的价值，这一点呢SystemServer是替代不了的，主要是因为SystemServer里跑了一堆系统服务，这些是不能继承到应用进程的。而且我们应用进程在启动的时候，内存空间除了必要的资源外，最好是干干净净的，不要继承一堆乱七八糟的东西。所以呢，不如给SystemServer和应用进程里都要用到的资源抽出来单独放在一个进程里，也就是这的zygote进程，然后zygote进程再分别孵化出SystemServer进程和应用进程。孵化出来之后，SystemServer进程和应用进程就可以各干各的事了。

zygote 和 Loop 为啥用socket 通信
	第一个原因，我们可以设想一下采用binder调用的话该怎么做，首先zygote要启用binder机制，需要打开binder驱动，获得一个描述符，再通过mmap进行内存映射，还要注册binder线程，这还不够，还要创建一个binder对象注册到serviceManager，另外AMS要向zygote发起创建应用进程请求的话，要先从serviceManager查询zygote的binder对象，然后再发起binder调用，这来来回回好几趟非常繁琐，相比之下，zygote和SystemServer进程本来就是父子关系，对于简单的消息通信，用管道或者socket非常方便省事。第二个原因，如果zygote启用binder机制，再fork出SystemServer，那么SystemServer就会继承了zygote的描述符以及映射的内存，这两个进程在binder驱动层就会共用一套数据结构，这显然是不行的，所以还得先给原来的旧的描述符关掉，再重新启用一遍binder机制，这个就是自找麻烦了。
	
总结：
	zygote 作用创建SystemServer 和孵化应用进程，Linux系统启动后Init进程加载配置文件，文件中配置要启动的服务 zygote就是其中之一 zygote启动后通过fork创建进程，进程创建后创建虚拟机 加载关键的Android 系统jni 方法，进入Java世界，预加载资源，启动SystemServe服务，准备Loop 。
 
 https://zhuanlan.zhihu.com/p/260414370
 
 ---
 
 说说Android系统启动流程？
 
	Android有哪些系统进程
	
	zygote怎么启动
	init 进程fork出zygote进程
	启动虚拟机，注册jni函数
	预加载系统资源
	启动SystemServer
	进入Sokect Looper []{E:\myNote\img\FrameWorke\Loop处理过程.png}
	 
	Looper循环中等到消息后 执行runOnce() 方法
	
	SytemServer 启动
	Zygote.forkSystemServer() 创建进程
	handleSystemServerProcess()
	RuntimeInit.zygoteInit() ｛
		commonInit();        常规初始化
		nativeZygoteInit()	启动binder机制binder线程
		applicationInit()    调用SystemServer man
	｝ 
	
system_server进程启动:
1、启动Boot服务
2、启动核心服务
3、启动其它服务
	
	run()         		1. 为主线程创建一个Loop    2. 加载共享库 android_servers  3. 创建系统上下文  4. 分别启动系统服务 5. 开启Loop循环  
	
	系统服务怎么启动的？
	怎么发布，让用户可见
	
	
	为什么系统服务不都跑在binder线程里？
	为什么系统服务不都跑在自己私有的工作线程里？
	跑在binder线程和跑在工作线程，如何取舍？
	
	怎么解决系统服务的相互依赖关系？
		1. 分批启动 
		2. 分阶段启动
		
SystemServer启动
	启动Binder机制  启动各类系统服务 进入Loop循环
--------------------

你知道怎么添加一个系统服务吗？

了解如何使用系统服务？

了解系统服务调用的基本原理？

了解服务的注册原理？

1. 添加系统服务的时机 
	如果在SystemServer进程中 可以在SystemServer进程启动的时候加入到启动系统服务中
	如果是单独进程服务，一方面要在 init.rc 中该下配置文件 另一方面 要注册到ServiceManager中 
	
2. 服务端要做哪些事
	启动binder机制 如果在SystemServer中不用考虑 SystemServer已经做好
	初始化
	给自己的binder注册到ServiceManager中

3. 应用端要做哪些事？ 
   同系统注册保持一致 注册ServiceFetcher 
   
---   
 系统服务和bind的应用服务有什么区别？  
	启动方式上？
		系统服务  systemServer 进程 或者 自己启动binder线程
		应用服务  启动加载在 应用端调用的
	注册方式上？
		系统服务  主动注册到SystemServer中
		应用服务  被动注册 应用请求服务时 AMS 请求binder 被动注册 []{E:\myNote\img\FrameWorke\应用服务的注册.png}
	使用方式上？
		服务：通过Context.getSystem() 
		应用：通过onServiceConnect 返回iBinder 
		
ServiceManger的启动和工作原理
	ServiceManger启动流程是怎样？
		启动进程
		启动Binder机制 
		发布自己的服务注册  向binder驱动注册
		进入looper循环等待并响应请求
	
	怎么获取ServiceManger的binder对象？
		根据0号
	怎么想ServiceManager添加服务？
		
	怎么从ServiceManger服务中获取服务？