### Glide初始化

##### Glide.with(Context)->getRetriever(Context)->Glide.get(Context).getRequestManagerRetriever()

1、get(Context)，获取注解的module，双检查锁初始化Glide->checkAndInitializeGlide(context, annotationGeneratedModule)

2、getRequestManagerRetriever()->RequestManagerRetriever

##### RequestManagerRetriever

1、通过HashMap来管理FragmentManager和RequestManagerFragment（SupportRequestManagerFragment）的映射



***

### Glide加载图片的过程

##### Glide.with(Context)->RequestManager

1、根据传入的上下文，查找对应上下文中的RequestManagerRetriever，再查找对应的RequestManager。

2、->RequestManagerRetriever.get(context)->获取FragmentManager->RequestManagerRetriever.supportFragmentGet()。从FramgentManger中获取SupportRequestManagerFragment。从SupportRequestManagerFragment获取RequestManager，如果没有找到，就创建一个，并绑定。

##### RequestManager

1、在RequestManager的构造方法中，RequestManager将自己加入SupportRequestManagerFragment的生命周期监听。

2、将自身加入到Glide的RequestManager列表

3、负责创建请求相关的RequestBuilder，并管理所有的Request

##### SupportRequestManagerFragment

1、在相应的FragmentManager中加入SupportRequestManagerFragment，并利用Fragment的生命周期来回调ActivityFragmentLifecycler中的生命钩子方法，从而控制RequestManager的请求

##### ActivityFragmentLifecycler

1、由SupportRequestManagerFragment持有，并由SupportRequestManagerFragment发起生命周期

2、提供生命周期的监听者注册接口



***

##### RequestManager.load()->RequestBuilder

1、asDrawable()：创建一个RequestBuilder对象，并返回

2、load()：绑定请求数据，并返回



***

##### Glide.apply(RequestOptions)->RequestBuilder

1、调用的是RequestBuilder.apply(RequestOptions)



***

##### Glide.into(View)

1、调用的是RequestBuilder.into(View)



##### RequestBuilder

1、into(View)中，执行下载方法->Request.begin()

2、判断各种状态，来处理

​	（1）model==null，直接onLoadFailed

​	（2）status==COMPLETE，则加载内存缓存MEMORY_CACHE

​	（3）status==WAITING_FOR_SIZE，如果尺寸还未测量，添加布局监听；尺寸已经测量完毕，或监听到尺寸测量完毕，都会进入onSizeReady方法

​	（4）onSizeReady方法，加锁，status==RUNNING，engine加载已经存在的图片或发起网络任务



##### Engine

1、load()->生成EngineKey->查看内存中是否存在缓存->存在则调用Request中的onResourceReady()加载资源；不存在则发起网络任务

2、waitForExistingOrStartNewJob()，先查看是否已经存在任务，不存在则新建任务



***

1、生命周期开始、暂停、结束时分别发生了什么？

答：（1）开始所有Request和Target

​	    （2）停止所有Request和Target

​	    （3）清理所有任务，移除监听、回调并取消注册

2、



***

### Glide加载、解析任务

1、load()->生成EngineKey->查看内存中是否存在缓存->存在则调用Request中的onResourceReady()加载资源；不存在则发起网络任务

2、waitForExistingOrStartNewJob()，先查看是否已经存在任务，不存在则新建任务

3、创建一个EngineJob、DecodeJob，EngineJob.start(DecodeJob)

##### EngineJob

1、负责选择合适的Engine来完成工作，DiskCacheExecutor或SourceExecutor

##### DecodeJob

1、runWrapped()，负责真正的过程执行

2、选择和状态对应的Generator

3、以SourceGenerator举例，Loader->Fetcher.loadData

4、HttpFetcher发起网络连接->调用Target.onResourceReady()

##### ImageViewTarget

1、onLoadStarted：设置占位图

2、onLoadFailed：设置错误图

3、onLoadCleared：设置占位图

4、onResourceReady：设置正确的资源

onResourceReady->DrawableImageViewTarget.setResource()->View.setImageDrawable(Drawable)



***

### Glide 缓存

##### 内存缓存

1、LruCache

2、