---
layout: post
title:  "Binder系列4—获取ServiceManager"
date:   2015-11-08 13:10:54
catalog:  true
tags:
    - android
    - binder

---
> 基于Android 6.0的源码剖析， 本文详细地讲解defaultServiceManager流程
	
	/framework/native/libs/binder/IServiceManager.cpp
	/framework/native/libs/binder/ProcessState.cpp
	/framework/native/libs/binder/BpBinder.cpp
	/framework/native/libs/binder/Binder.cpp

	/framework/native/include/binder/IServiceManager.h
	/framework/native/include/binder/IInterface.h


## 概述

获取Service Manager是通过`defaultServiceManager()`方法来完成，当进程[注册服务(addService)](http://gityuan.com/2015/11/14/binder-add-service/)或
[获取服务(getService)](http://gityuan.com/2015/11/15/binder-get-service/)的过程之前，都需要先调用defaultServiceManager()方法来获取`gDefaultServiceManager`对象。对于gDefaultServiceManager对象，如果存在则直接返回；如果不存在则创建该对象，创建过程包括调用open()打开binder驱动设备，利用mmap()映射内核的地址空间。

点击查看[大图](http://gityuan.com/images/binder/get_servicemanager/get_servicemanager.jpg)

![get_servicemanager](/images/binder/get_servicemanager/get_servicemanager.jpg)

## 1. defaultServiceManager
==> `/framework/native/libs/binder/IServiceManager.cpp`

	sp<IServiceManager> defaultServiceManager()
	{
	    if (gDefaultServiceManager != NULL) return gDefaultServiceManager;    
	    {
	        AutoMutex _l(gDefaultServiceManagerLock); //加锁
	        while (gDefaultServiceManager == NULL) {
                 //【见流程2、5、9】
	            gDefaultServiceManager = interface_cast<IServiceManager>(
	                ProcessState::self()->getContextObject(NULL)); 
	            if (gDefaultServiceManager == NULL)
	                sleep(1);   //休眠1秒，往往在系统刚启动过程可能会第一次获取失败
	        }
	    }
	    return gDefaultServiceManager;
	}
  
获取ServiceManager对象采用**单例模式**，当gDefaultServiceManager存在，则直接返回，否则创建一个新对象。 发现与一般的单例模式不太一样，里面多了一层while循环，这是google在2013年1月Todd Poynor提交的修改。当尝试创建或获取ServiceManager时，ServiceManager可能尚未准备就绪，这时通过sleep 1秒后，循环尝试获取直到成功。


`defaultServiceManager()`最为核心的代码：

	interface_cast<IServiceManager>(ProcessState::self()->getContextObject(NULL));  

分解为以下3个步骤：

- `ProcessState::self()`：用于获取ProcessState对象(也是单例模式)，每个进程有且只有一个ProcessState对象，存在则直接返回，不存在则创建，详情见【流程2~4】;
- `getContextObject()`： 用于获取BpBiner对象，对于handle=0的BpBiner对象，存在则直接返回，不存在才创建，详情见【流程5~8】;
- `interface_cast<IServiceManager>()`：创建BpServiceManager对象，详情见【流程9~11】.


## 2. ProcessState::self
==> `/framework/native/libs/binder/ProcessState.cpp`

获得ProcessState对象

	sp<ProcessState> ProcessState::self()
	{
	    Mutex::Autolock _l(gProcessMutex);
	    if (gProcess != NULL) {
	        return gProcess;
	    }

	    //实例化ProcessState 【见流程3】
	    gProcess = new ProcessState;
	    return gProcess;
	}


这也是**单例模式**，从而保证每一个进程只有一个`ProcessState`对象。其中`gProcess`和`gProcessMutex`是保存在`Static.cpp`类的全局变量。

## 3. New ProcessState
==> `/framework/native/libs/binder/ProcessState.cpp`

初始化ProcessState对象 

	ProcessState::ProcessState()
	    : mDriverFD(open_driver()) // 打开Binder驱动【见流程4】
	    , mVMStart(MAP_FAILED)
	    , mThreadCountLock(PTHREAD_MUTEX_INITIALIZER)     
	    , mThreadCountDecrement(PTHREAD_COND_INITIALIZER) 
	    , mExecutingThreadsCount(0)                      
	    , mMaxThreads(DEFAULT_MAX_BINDER_THREADS)       
	    , mManagesContexts(false)
	    , mBinderContextCheckFunc(NULL)
	    , mBinderContextUserData(NULL)
	    , mThreadPoolStarted(false)
	    , mThreadPoolSeq(1)
	{
	    if (mDriverFD >= 0) {
	        //采用内存映射函数mmap，给binder分配一块虚拟地址空间,用来接收事务
	        mVMStart = mmap(0, BINDER_VM_SIZE, PROT_READ, MAP_PRIVATE | MAP_NORESERVE, mDriverFD, 0);
	        if (mVMStart == MAP_FAILED) {
	            close(mDriverFD); //没有足够空间分配给/dev/binder,则关闭驱动
	            mDriverFD = -1;
	        }
	    }
	}

 

- `ProcessState`的单例模式的惟一性，因此一个进程只打开binder设备一次,其中ProcessState的成员变量`mDriverFD`记录binder驱动的fd，用于访问binder设备。
- `BINDER_VM_SIZE = (1*1024*1024) - (4096 *2)`, binder分配的默认内存大小为1M-8k。
- `DEFAULT_MAX_BINDER_THREADS = 15`，binder默认的最大可并发访问的线程数为16。

## 4. open_driver
==> `/framework/native/libs/binder/ProcessState.cpp`

打开Binder驱动设备

	static int open_driver()
	{
	    // 打开/dev/binder设备，建立与内核的Binder驱动的交互通道
	    int fd = open("/dev/binder", O_RDWR); 
	    if (fd >= 0) {
	        fcntl(fd, F_SETFD, FD_CLOEXEC);
	        int vers = 0;
	        status_t result = ioctl(fd, BINDER_VERSION, &vers);
	        if (result == -1) {
	            close(fd);
	            fd = -1;
	        }
	        if (result != 0 || vers != BINDER_CURRENT_PROTOCOL_VERSION) {
	            close(fd);
	            fd = -1;
	        }
	        size_t maxThreads = DEFAULT_MAX_BINDER_THREADS;
	
	        // 通过ioctl设置binder驱动，能支持的最大线程数
	        result = ioctl(fd, BINDER_SET_MAX_THREADS, &maxThreads);
	        if (result == -1) {
	            ALOGE("Binder ioctl to set max threads failed: %s", strerror(errno));
	        }
	    } else {
	        ALOGW("Opening '/dev/binder' failed: %s\n", strerror(errno));
	    }
	    return fd;
	}

open_driver作用是打开/dev/binder设备，设定binder支持的最大线程数。关于binder驱动的相应方法，见文章[Binder Driver初探](http://gityuan.com/2015/11/01/binder-driver/)。


## 5. getContextObject
==> `/framework/native/libs/binder/ProcessState.cpp`

获取handle=0的IBinder

	sp<IBinder> ProcessState::getContextObject(const sp<IBinder>& /*caller*/)
	{
	    return getStrongProxyForHandle(0);  //【见流程6】
	}



## 6. getStrongProxyForHandle
==> `/framework/native/libs/binder/ProcessState.cpp`

获取IBinder

	sp<IBinder> ProcessState::getStrongProxyForHandle(int32_t handle)
	{
	    sp<IBinder> result;
	
	    AutoMutex _l(mLock);
	    //查找handle对应的资源项【见流程7】
	    handle_entry* e = lookupHandleLocked(handle); 
	
	    if (e != NULL) {
	        IBinder* b = e->binder;
	        if (b == NULL || !e->refs->attemptIncWeak(this)) {
	            if (handle == 0) {
	                Parcel data;
                    //通过ping操作测试binder是否准备就绪
	                status_t status = IPCThreadState::self()->transact(
	                        0, IBinder::PING_TRANSACTION, data, NULL, 0); 
	                if (status == DEAD_OBJECT)
	                   return NULL;
	            }
	            //当handle值所对应的IBinder不存在或弱引用无效时，则创建BpBinder对象【见流程8】
	            b = new BpBinder(handle); 
	            e->binder = b;
	            if (b) e->refs = b->getWeakRefs();
	            result = b;
	        } else {
	            result.force_set(b);
	            e->refs->decWeak(this);
	        }
	    }	
	    return result;
	}

当handle值所对应的IBinder不存在或弱引用无效时会创建BpBinder，否则直接获取。  
针对handle==0的特殊情况，通过PING_TRANSACTION来判断是否准备就绪。如果在context manager还未生效前，一个BpBinder的本地引用就已经被创建，那么驱动将无法提供context manager的引用。

## 7. lookupHandleLocked
==> `/framework/native/libs/binder/ProcessState.cpp`

	ProcessState::handle_entry* ProcessState::lookupHandleLocked(int32_t handle)
	{
	    const size_t N=mHandleToObject.size();
	    //当handle大于mHandleToObject的长度时，进入该分支
	    if (N <= (size_t)handle) {
	        handle_entry e;
	        e.binder = NULL;
	        e.refs = NULL;
	        //从mHandleToObject的第N个位置开始，插入(handle+1-N)个e到队列中
	        status_t err = mHandleToObject.insertAt(e, N, handle+1-N);
	        if (err < NO_ERROR) return NULL;
	    }
	    return &mHandleToObject.editItemAt(handle);
	}

根据handle值来查找对应的`handle_entry`,`handle_entry`是一个结构体，里面记录IBinder和weakref_type两个指针。当handle大于mHandleToObject的Vector长度时，则向该Vector中添加(handle+1-N)个handle_entry结构体，然后再返回handle向对应位置的handle_entry结构体指针。

## 8. new BpBinder
==> `/framework/native/libs/binder/BpBinder.cpp`

创建BpBinder对象

	BpBinder::BpBinder(int32_t handle)
	    : mHandle(handle)
	    , mAlive(1)
	    , mObitsSent(0)
	    , mObituaries(NULL)
	{
	    extendObjectLifetime(OBJECT_LIFETIME_WEAK); //延长对象的生命时间
	    IPCThreadState::self()->incWeakHandle(handle); //handle所对应的bindle弱引用 + 1
	}

创建BpBinder对象中，会将handle相对应Binder的弱引用增加1.


## 9. interface_cast
==> `/framework/native/include/binder/IInterface.h`

interface_cast，这是一个模板函数，如下：

	template<typename INTERFACE>
	inline sp<INTERFACE> interface_cast(const sp<IBinder>& obj)
	{
	    return INTERFACE::asInterface(obj); //【见流程10】
	}

可以得出，`interface_cast<IServiceManager>()` 等价于 `IServiceManager::asInterface()`。接下来,再来说说`asInterface()`函数的具体功能。


## 10. IServiceManager::asInterface

对于asInterface()函数，通过搜索代码，你会发现根本找不到这个方法是在哪里定义这个函数的，其实跟前面小节9的方式类似，也是通过模板函数来定义的，通过下面两个代码完成的：

    //位于IServiceManager.h文件
    DECLARE_META_INTERFACE(IServiceManager) 
    //位于IServiceManager.cpp文件
    IMPLEMENT_META_INTERFACE(ServiceManager,"android.os.IServiceManager")

接下来，再说说这两行代码分别完成的功能：

**（1） DECLARE_META_INTERFACE(IServiceManager)**  
==> `/framework/native/include/binder/IServiceManager.h`  
	
根据`IInterface.h`中的模板函数，展开即可得：
	
	static const android::String16 descriptor;		 

	static android::sp< IServiceManager > asInterface(const android::sp<android::IBinder>& obj)	 

	virtual const android::String16& getInterfaceDescriptor() const;	 

	IServiceManager ();                                                   
	virtual ~IServiceManager();

**（2） IMPLEMENT_META_INTERFACE(ServiceManager,"android.os.IServiceManager")**   
==> `/framework/native/libs/binder/IServiceManager.cpp`


根据`IInterface.h`中的模板函数，展开即可得：

	const android::String16 IServiceManager::descriptor(“android.os.IServiceManager”);

	const android::String16& IServiceManager::getInterfaceDescriptor() const
	{ 
	     return IServiceManager::descriptor;
	}    
	
	 android::sp<IServiceManager> IServiceManager::asInterface(const android::sp<android::IBinder>& obj)
	{
	       android::sp<IServiceManager> intr;
	        if(obj != NULL) {                                              
	           intr = static_cast<IServiceManager *>(                         
	               obj->queryLocalInterface(IServiceManager::descriptor).get());  
	           if (intr == NULL) {
	               intr = new BpServiceManager(obj);  //【见流程11】
	            }
	        }
	       return intr;
	}

	IServiceManager::IServiceManager () { }
	IServiceManager::~ IServiceManager() { }

不难发现，`IServiceManager::asInterface()` 等价于 `new BpServiceManager()`。在这里，更确切地说应该是new BpServiceManager(BpBinder)。


## 11. new BpServiceManager

创建BpServiceManager对象的过程，会先初始化父类对象：

**（1）初始化BpServiceManager**  
==> `/framework/native/libs/binder/IServiceManager.cpp`

    BpServiceManager(const sp<IBinder>& impl)
        : BpInterface<IServiceManager>(impl)
    {
    }

**（2）初始化父类BpInterface**  
==> `/framework/native/include/binder/IInterface.h`  

	inline BpInterface<INTERFACE>::BpInterface(const sp<IBinder>& remote)
	    :BpRefBase(remote)
	{
	}

**（3）初始化父类BpRefBase**  
==> `/framework/native/libs/binder/Binder.cpp`    

	BpRefBase::BpRefBase(const sp<IBinder>& o)
	    : mRemote(o.get()), mRefs(NULL), mState(0)
	{
	    extendObjectLifetime(OBJECT_LIFETIME_WEAK);
	
	    if (mRemote) {
	        mRemote->incStrong(this);           
	        mRefs = mRemote->createWeak(this);  
	    }
	}

`new BpServiceManager()`，在初始化过程中，比较重要工作的是类BpRefBase的mRemote指向new BpBinder(0)，从而BpServiceManager能够利用Binder进行通过通信。


## 12. 小结

- defaultServiceManager 等价于
	
		sp<IServiceManager> sm = new BpServiceManager(new BpBinder(0));

- ProcessState::self()主要工作：
	1. 调用open()，打开/dev/binder驱动设备；
	2. 再利用mmap()，创建大小为`1M-8K`的内存地址空间；
	3. 设定当前进程最大的最大并发Binder线程个数为`16`。
	
- BpServiceManager巧妙将通信层与业务层逻辑合为一体，
	1. 通过继承接口`IServiceManager`实现了接口中的业务逻辑函数；
	2. 通过成员变量`mRemote`= new BpBinder(0)进行Binder通信工作。 
	 
- BpBinder通过handler来指向所对应BBinder, 在整个Binder系统中`handle=0`代表ServiceManager所对应的BBinder。